# ভালো Dockerfile এবং Docker Compose লেখার গাইড 🐳

## Part 1: ভালো Dockerfile কীভাবে লিখবে

### প্রথমে বোঝো — Docker Image কীভাবে Build হয়

Dockerfile এর প্রতিটা **Line** মানেই হলো একটি **Layer**। এই Layer গুলো একটির ওপর একটি স্ট্যাক (Stack) হয়ে চূড়ান্ত **Image** তৈরি করে। ডকার ক্যাশ মেকানিজম ব্যবহার করে যাতে পরেরবার বিল্ড করার সময় সময় কম লাগে।

| কমান্ড | স্তরের বিবরণ (Layer Description) |
| :--- | :--- |
| `FROM node:20-alpine` | **Layer 1:** বেস ইমেজ সেট করে (Lightweight Linux + Node.js) |
| `WORKDIR /app` | **Layer 2:** কন্টেইনারের ভেতরে কাজের জায়গা (Directory) তৈরি করে |
| `COPY package*.json ./` | **Layer 3:** শুধুমাত্র ডিপেন্ডেন্সি লিস্ট কপি করে (ক্যাশিং সুবিধার জন্য) |
| `RUN npm install` | **Layer 4:** প্রয়োজনীয় প্যাকেজগুলো কন্টেইনারে ইনস্টল করে |
| `COPY . .` | **Layer 5:** আপনার প্রজেক্টের বাকি সব কোড কন্টেইনারে নিয়ে আসে |

---

> **Pro Tip:** আমরা কেন `COPY . .` করার আগেই `npm install` করি?
> কারণ, আপনার কোড বারবার চেঞ্জ হলেও `package.json` সচরাচর চেঞ্জ হয় না। ফলে ডকার ৩ নম্বর লেয়ার পর্যন্ত ক্যাশ ব্যবহার করে বিল্ড প্রসেসকে অনেক ফাস্ট করে দেয়!

### সবচেয়ে গুরুত্বপূর্ণ Concept: Cache ⚡

ডকার প্রতিটি লেয়ারের একটি **Hash** তৈরি করে রাখে। যদি কোনো লেয়ারে পরিবর্তন না আসে, তবে ডকার আগের বিল্ডের রেজাল্ট (Cache) ব্যবহার করে।

* **Layer 3 change হলে** (অর্থাৎ `package.json` এ নতুন লাইব্রেরি যোগ করলে) → ডকার ৩ নম্বর লেয়ার এবং তার নিচের সব লেয়ার (৪ ও ৫) নতুন করে **Rebuild** করবে।
* **Layer 3 same থাকলে** → ডকার ৩ ও ৪ নম্বর লেয়ার সরাসরি **Cache** থেকে নেয়। ✅

#### এজন্যই আমরা `package.json` আগে কপি করি:
আমরা সাধারণত কোড (Layer 5) প্রতি মিনিটে শতবার চেঞ্জ করি, কিন্তু `package.json` (Layer 3) খুব কম চেঞ্জ করি। ফলে আপনি প্রতিবার কোড চেঞ্জ করে বিল্ড দিলেও ডকারকে আর `npm install` চালাতে হয় না, যা আপনার বিল্ড টাইম **১০ মিনিট থেকে কমিয়ে ১০ সেকেন্ডে** নিয়ে আসতে পারে!

---

#### লেয়ার অপ্টিমাইজেশন চিত্র:
1. `FROM node:20-alpine` (Rarely changes)
2. `WORKDIR /app` (Never changes)
3. `COPY package*.json ./` (Changes only when adding packages)
4. `RUN npm install` (**Heavy task!** - Cached if Step 3 is same)
5. `COPY . .` (Changes often - Always rebuilds this layer)

### ❌ খারাপ Dockerfile
```bash
FROM node:18

COPY . .

RUN npm install

EXPOSE 3000

CMD ["node", "index.js"]
```

**সমস্যাগুলো:**
```ts
১. node:18 → Full image, ~1GB! alpine হলে ~150MB
২. COPY . . আগে → code change হলে npm install আবার চলবে
৩. node_modules ও copy হবে (host এর পুরানো modules)
৪. .dockerignore নেই → অপ্রয়োজনীয় files ঢুকছে
৫. USER নেই → root হিসেবে চলছে (security risk)
```

### ✅ ভালো Dockerfile — Node.js (Standard Practice)

```dockerfile
# ── Stage: Base Image ──────────────────────────
# alpine = ছোট Linux, image size অনেক কমে
# নির্দিষ্ট version দাও, "latest" দিও না
FROM node:20-alpine

# ── Working Directory ──────────────────────────
# সব files এখানে থাকবে, root এ ছড়াবে না
WORKDIR /app

# ── Dependencies আগে Copy করো ──────────────────
# এই দুটো file আলাদা copy করার কারণ:
# package.json না বদলালে npm install cache থেকে নেবে
# তাই build অনেক fast হবে
COPY package*.json ./

# ── Dependencies Install ────────────────────────
# npm ci → npm install এর চেয়ে ভালো production এ
# ci = clean install, package-lock.json exactly follow করে
RUN npm ci

# ── Source Code Copy ────────────────────────────
# এটা সবার শেষে কারণ code বেশি change হয়
# উপরের layers cache থাকবে
COPY . .

# ── Port Document করো ──────────────────────────
# এটা শুধু documentation, actual mapping compose এ
EXPOSE 3000

# ── Container Start Command ─────────────────────
# Array format use করো, string না
# String হলে shell এর মধ্যে দিয়ে চলে (overhead)
CMD ["node", "index.js"]
```

### ✅ ভালো Dockerfile — React/Vite (Development)

```bash
FROM node:20-alpine

WORKDIR /app

# package.json আগে
COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5173

# vite --host না দিলে container বাইরে accessible হবে না
CMD ["npm", "run", "dev"]
```

### ✅ Multi-stage Dockerfile — Production React
Production এ Vite dev server চালানো উচিত না। Build করে nginx দিয়ে serve করতে হয়।
