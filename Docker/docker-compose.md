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

```bash
# ── Stage 1: Build ─────────────────────────────
# এই stage এ node থাকবে, শুধু build করার জন্য
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .

# React app build করো
RUN npm run build
# /app/dist এ static files তৈরি হবে

# ── Stage 2: Serve ─────────────────────────────
# এই stage এ node নেই, শুধু nginx
# Final image অনেক ছোট হবে (~25MB vs ~400MB)
FROM nginx:alpine

# Build হওয়া files nginx এর folder এ copy করো
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```
```
Stage 1 (builder): node:20-alpine + সব dependencies = ~400MB
Stage 2 (final):   nginx:alpine + শুধু dist files = ~25MB

Final image এ node থাকে না! শুধু built files থাকে।
```

---

### .dockerignore — অবশ্যই লিখবে

`.dockerignore` ছাড়া `COPY . .` করলে অপ্রয়োজনীয় সব কিছু image এ ঢুকবে।
```
# backend/.dockerignore
node_modules        ← host এর modules, container এ দরকার নেই
.env                ← secret, image এ রাখা বিপজ্জনক
.git                ← git history দরকার নেই
*.log               ← log files দরকার নেই
.DS_Store           ← Mac এর system file
README.md           ← documentation দরকার নেই
```
```
# frontend/.dockerignore
node_modules
dist                ← আগের build, container এ দরকার নেই
.env
.git
*.log
```

`.dockerignore` থাকলে:
```
Build fast হয়      ← কম files transfer হয়
Image ছোট হয়       ← অপ্রয়োজনীয় files নেই
Secure হয়          ← .env, secrets image এ যায় না
```

---

### Dockerfile লেখার Rules মনে রাখো
```
Rule 1: সবসময় নির্দিষ্ট version দাও
  ❌ FROM node:latest    → কাল update হলে সব ভাঙতে পারে
  ✅ FROM node:20-alpine → সবসময় same

Rule 2: alpine use করো যেখানে পারো
  ❌ FROM node:20        → ~1GB
  ✅ FROM node:20-alpine → ~150MB

Rule 3: package.json আগে copy করো
  ❌ COPY . .
     RUN npm install    → code change = npm install আবার

  ✅ COPY package*.json ./
     RUN npm install
     COPY . .           → code change = npm install skip ✅

Rule 4: .dockerignore অবশ্যই লিখবে
  node_modules, .env, .git সবসময় ignore

Rule 5: CMD এ array format
  ❌ CMD "node index.js"
  ✅ CMD ["node", "index.js"]

Rule 6: Production এ npm ci
  ❌ RUN npm install     → package-lock.json ignore করতে পারে
  ✅ RUN npm ci          → exact versions install করে
```

## Part 2: ভালো docker-compose.yml কীভাবে লিখবে

```bash
services:
  service_name:
    # ১. Image source (image বা build)
    # ২. container_name
    # ৩. ports
    # ৪. environment / env_file
    # ৫. volumes
    # ৖. depends_on
    # ৭. networks
    # ৮. restart

volumes:

networks:
```

## ❌ খারাপ docker-compose.yml

```bash
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_PASSWORD=supersecret123
    restart: always
```

**সমস্যাগুলো:**
```
১. Password সরাসরি লেখা → GitHub এ push হলে সবাই দেখবে
২. network নেই → default network use করবে (DNS কাজ করবে না)
৩. volume নেই → data হারাবে
৪. depends_on নেই → DB ready হওয়ার আগে app connect করবে
৫. healthcheck নেই → container চলছে মানে app ready না
```

## ✅ ভালো docker-compose.yml

```bash
# version obsolete, না লিখলেই ভালো
# কিন্তু লিখলেও কাজ করে, warning দেখায় শুধু

services:

  # ── Database ────────────────────────────────
  mongodb:
    image: mongo:6                    # সবসময় version দাও
    container_name: todo-mongodb

    # Secret কখনো এখানে লিখবে না
    # .env file থেকে নাও
    env_file:
      - .env

    volumes:
      - mongodata:/data/db            # data persist করার জন্য

    networks:
      - todo-net

    restart: unless-stopped           # manually বন্ধ না হলে সবসময় চলবে

  # ── Backend ─────────────────────────────────
  backend:
    build:
      context: ./backend              # কোন folder
      dockerfile: Dockerfile          # কোন Dockerfile (নাম আলাদা হলে)

    container_name: todo-backend

    ports:
      - "5000:5000"

    # Secrets .env থেকে নাও
    env_file:
      - .env

    # Non-secret config এখানে দিতে পারো
    environment:
      - NODE_ENV=development

    volumes:
      - ./backend:/app                # bind mount (dev এ)
      - /app/node_modules             # node_modules রক্ষা করো

    depends_on:
      - mongodb

    networks:
      - todo-net

    restart: on-failure

  # ── Frontend ────────────────────────────────
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile

    container_name: todo-frontend

    ports:
      - "5173:5173"

    volumes:
      - ./frontend:/app
      - /app/node_modules

    depends_on:
      - backend

    networks:
      - todo-net

    restart: on-failure

# ── Volumes ─────────────────────────────────────
volumes:
  mongodata:                          # Docker manage করবে

# ── Networks ────────────────────────────────────
networks:
  todo-net:
    driver: bridge                    # default bridge, explicitly লিখলে clear হয়

```

## .env file — Secret আলাদা রাখো

```bash
# .env
MONGO_INITDB_ROOT_USERNAME=admin
MONGO_INITDB_ROOT_PASSWORD=supersecret123
MONGO_URL=mongodb://admin:supersecret123@mongodb:27017/tododb
PORT=5000
NODE_ENV=development
```


```bash
# .gitignore — এটা অবশ্যই করবে
.env
node_modules
```
```
.env → local এ থাকবে
.gitignore → GitHub এ যাবে না
কেউ তোমার repo দেখলে password দেখবে না ✅
```

---

### docker-compose লেখার Rules মনে রাখো
```
Rule 1: Secret কখনো compose file এ লিখবে না
  ❌ environment:
       - DB_PASSWORD=secret123
  ✅ env_file:
       - .env

Rule 2: .env কখনো GitHub এ push করবে না
  .gitignore এ .env লিখবেই

Rule 3: সবসময় custom network দাও
  Default network এ DNS কাজ করে না
  Custom network এ service name দিয়ে connect হয়

Rule 4: Database এ সবসময় volume দাও
  Volume ছাড়া container delete = data delete

Rule 5: Image এ সবসময় version tag দাও
  ❌ image: mongo
  ✅ image: mongo:6

Rule 6: depends_on মানে "ready" না, "started"
  Database connect এ retry logic রাখো code এ

Rule 7: Indentation সবসময় 2 space
  Tab দিলে error আসে YAML এ
```

---

## Part 3: তোমার Error গুলোর ব্যাখ্যা এবং ভবিষ্যতে কী করবে

---

### Error 1: External Drive Bind Mount
```
The path /media/awolad/MEDIA1.1/todo-app/backend
is not shared from the host
```

**কারণ:**
Docker bind mount শুধু নির্দিষ্ট কিছু folder access করতে পারে। Linux এ default হলো `/home`। External drive `/media/...` এ Docker এর permission নেই।

**ভবিষ্যতে মনে রাখবে:**
```
Docker project সবসময় home directory তে রাখো
✅ ~/myproject
✅ /home/awolad/myproject
❌ /media/...
❌ /mnt/...
❌ /tmp/...
```

---

### Error 2: Node Version Mismatch
```
You are using Node.js 18.20.8.
Vite requires Node.js version 20.19+ or 22.12+
TypeError: crypto.hash is not a function
```

**কারণ:**
নতুন Vite এর কিছু feature Node 20 এ introduce হয়েছে। Node 18 এ `crypto.hash` function নেই।

**ভবিষ্যতে মনে রাখবে:**
```
নতুন package install করার আগে Node version requirement দেখো
package এর docs বা npmjs.com এ লেখা থাকে

Dockerfile এ সবসময় নির্দিষ্ট version দাও:
✅ FROM node:20-alpine   (Vite এর জন্য)
✅ FROM node:18-alpine   (পুরানো projects এর জন্য)
❌ FROM node:latest      (আজ কাজ করে, কাল নাও করতে পারে)
```

---

### Error 3: Docker Cache
```
CACHED [forntend 4/5] RUN npm install
← Dockerfile change করলেও পুরানো cache use করছে
```
কারণ:
Docker প্রতিটা layer cache করে রাখে। Image delete না করলে বা --no-cache না দিলে পুরানো cache use করে। তাই Dockerfile change করলেও reflect হয় না।

## ভবিষ্যতে মনে রাখবে:
```bash
# Dockerfile change করলে এটা করো:
docker compose down
docker rmi image_name          # পুরানো image delete
docker compose up --build      # rebuild

# অথবা নিশ্চিত হতে:
docker compose build --no-cache    # cache ছাড়া build
docker compose up
```

Cache কখন problem হয়:
```
✅ Cache ভালো → Code change, npm install skip → Fast build
❌ Cache সমস্যা → Dockerfile change করলে reflect হয় না
```

---

### Error 4: `version` Obsolete Warning
```
the attribute `version` is obsolete,
it will be ignored, please remove it
```
কারণ:
নতুন Docker Compose এ version field আর দরকার নেই। Automatically detect করে।

