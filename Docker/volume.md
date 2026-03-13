# Volume কী?

Volume হলো Docker এর একটা storage mechanism যেটা container এর বাইরে data রাখে।


## প্রথমে বোঝো - Problem টা কী?

Docker container হলো temporary। মানে:
```ts
Container চালু → Data তৈরি হলো → Container delete → Data গেলো 💀
```

## Real example বোঝো:
- তুমি একটা MongoDB container চালালে
- কিছু data insert করলে
- docker rm দিলে
- আবার container চালালে → সব data গেছে!

এই সমস্যার সমাধান হলো Volume

```ts
Without Volume:
[Container] → data container এর ভেতরে → container গেলে data যায়

With Volume:
[Container] ←→ [Volume] → Volume আলাদাভাবে থাকে → container গেলেও data থাকে
```

Docker Volume আসলে তোমার host machine এ একটা জায়গায় data রাখে।
`Linux এ: /var/lib/docker/volumes/`

## Volume এর ৩ টা Type আছে
```bash
1. Named Volume      → Docker manage করে, নাম দিয়ে চেনা যায়
2. Bind Mount        → তোমার নিজের folder map করো
3. Anonymous Volume  → Docker manage করে, নাম নেই (avoid করো)
```

## Type 1: Named Volume

```bash
docker volume create mydata
           ↓
Docker নিজে একটা storage area তৈরি করে
           ↓
Container এর কোনো folder এর সাথে connect করো
           ↓
Container delete হলেও "mydata" volume থেকে যায়
```

## Step by Step Practice

*Step 1: একটা Volume তৈরি করো*

```bash
docker volume create mydata
```

Output আসবে:
```
mydata
```
*Step 2: Volume list দেখো*

```bash
docker volume ls
```

Output:
```
DRIVER    VOLUME NAME
local     mydata
```
*Step 3: Volume এর details দেখো*

```bash
docker volume inspect mydata
```
**Output**

```bash
[
    {
        "CreatedAt": "2026-03-01T08:49:27Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mydata/_data",
        "Name": "mydata",
        "Options": null,
        "Scope": "local"
    }
]
```
`Mountpoint` মানে host machine এ এখানে data আসলে save হচ্ছে।

*Step 4: Volume ছাড়া সমস্যা দেখো (Prove করো)*

আগে দেখো volume ছাড়া data কীভাবে হারায়:

```bash
# একটা ubuntu container চালাও
docker run -it --name test1 ubuntu bash
```

*Container এর ভেতরে এখন:*
```bash
# একটা file তৈরি করো
echo "আমার important data" > /tmp/myfile.txt

# দেখো file আছে
cat /tmp/myfile.txt

# container থেকে বেরিয়ে আসো
exit
```

```bash
# container delete করো
docker rm test1

# আবার নতুন container চালাও
docker run -it --name test2 ubuntu bash
```

*Container এর ভেতরে:*

```bash
# file খোঁজো
cat /tmp/myfile.txt
# Output: No such file or directory 💀
exit
```
Data গেছে! এটাই সমস্যা।

*Step 5: এখন Volume দিয়ে Solve করো*

```bash
# Volume সহ container চালাও
# Format: -v volume_name:container_path
docker run -it --name test3 -v mydata:/myapp/data ubuntu bash
```

`-v mydata:/myapp/data` মানে:
```
mydata (volume) ←→ /myapp/data (container এর এই folder)
এই দুইটা এখন একই জায়গায় point করছে
```
*Container এর ভেতরে:*

```bash
# data লিখো
echo "এই data থাকবে সবসময়" > /myapp/data/important.txt
cat /myapp/data/important.txt

exit
```

```bash
# container delete করো
docker rm test3

# নতুন container চালাও, same volume দাও
docker run -it --name test4 -v mydata:/myapp/data ubuntu bash
```

*Container এর ভেতরে:*

```bash
cat /myapp/data/important.txt
# Output: এই data থাকবে সবসময় ✅
exit
```
- Data টিকে গেছে! Container গেলেও volume এ data আছে।

**Step 6: Volume Delete করো**

```bash
# volume delete (container চলতে থাকলে delete হবে না)
docker volume rm mydata

# সব unused volumes একসাথে delete
docker volume prune
```

# Type 2: Bind Mount

## Concept

Named Volume এ Docker নিজে ঠিক করে data কোথায় রাখবে।
**Bind Mount** এ তুমি নিজে বলো কোন folder use করবে।

``` bash
তোমার PC তে: /home/user/myproject
                    ↕️ (mirror)
Container এ:  /app

তুমি /home/user/myproject এ file change করলে
Container এর /app এও সাথে সাথে change হবে
```

- এটা কখন লাগে?
Development এর সময়! Code change করলে সাথে সাথে container এ reflect হবে, আবার build করতে হবে না।


## Step by Step Practice

**Step 1: একটা Project Folder তৈরি করো**

```bash
mkdir ~/docker-practice
cd ~/docker-practice
```

**Step 2: একটা Simple File তৈরি করো**
```bash
echo "Hello from host machine!" > message.txt
ls
# message.txt দেখাবে
```

**Step 3: Bind Mount দিয়ে Container চালাও**
```bash
# Format: -v /host/path:/container/path
docker run -it --name bindtest -v $(pwd):/myapp ubuntu bash
```

- `$(pwd)` মানে current directory (যেখানে তুমি এখন আছো)

**Container এর ভেতরে:**

```bash
cd /myapp
ls
# message.txt দেখাবে ✅

cat message.txt
# Hello from host machine!
```

**Step 4: Container থেকে File তৈরি করো**
Container এর ভেতরে থেকে:
```bash
echo "Container থেকে লেখা" > from-container.txt
exit
```
এখন host machine এ দেখো:

```bash
ls ~/docker-practice
# message.txt এবং from-container.txt দুটোই আছে ✅
cat from-container.txt
# Container থেকে লেখা
```
- দুইদিক থেকেই কাজ করে!

### Step 5: Live Code Reload দেখো (Real Use Case)
এটাই Bind Mount এর আসল power। একটা Node.js example:

```bash
mkdir ~/node-live
cd ~/node-live
``

- index.js file তৈরি করো:

```bash
// index.js
const http = require('http');

const server = http.createServer((req, res) => {
  res.end('Version 1 - পরিবর্তন দেখো!\n');
});

server.listen(3000, () => console.log('Running on 3000'));
```

- Container চালাও:

```bash
docker run -d \
  --name liveapp \
  -p 3000:3000 \
  -v $(pwd):/app \
  -w /app \
  node:18-alpine \
  node index.js
```
- `-w /app` মানে `working directory /app সেট করো।

```bash
Browser এ বা terminal এ দেখো:
```

```bash
curl localhost:3000
# Version 1 - পরিবর্তন দেখো!
```
- এখন index.js এ change করো (container বন্ধ না করে):

```bash
res.end('Version 2 - আমি বদলে গেছি!\n');
```
- Container restart করো (nodemon ছাড়া manually):

```bash
docker restart liveapp
curl localhost:3000
# Version 2 - আমি বদলে গেছি! ✅
```

---

# Named Volume vs Bind Mount - পার্থক্য
```
Named Volume:
✅ Production এ ব্যবহার করো
✅ Database data রাখতে (MySQL, MongoDB, PostgreSQL)
✅ Docker নিজে manage করে, safe
❌ সরাসরি files edit করা কঠিন

Bind Mount:
✅ Development এ ব্যবহার করো
✅ Code live reload করতে
✅ Host এর files সরাসরে access
❌ Production এ risky (host এর যেকোনো file access হতে পারে)
```
## সব Volume Commands এক জায়গায়

```bash
# Volume তৈরি
docker volume create myvolume

# সব volumes দেখো
docker volume ls

# Volume এর details
docker volume inspect myvolume

# Volume delete
docker volume rm myvolume

# সব unused volumes delete
docker volume prune

# Named volume দিয়ে run
docker run -v myvolume:/app/data myimage

# Bind mount দিয়ে run
docker run -v /host/path:/container/path myimage
docker run -v $(pwd):/app myimage          # current folder
```

---

# Quick Summary
```
Data রাখতে হবে permanently?  → Named Volume ব্যবহার করো
Development এ code sync?      → Bind Mount ব্যবহার করো
Container delete = Data delete → এটা avoid করতেই Volume
```
___

## Practice

```js
const express = require('express');
const fs = require('fs');
const app = express();

app.use(express.json());

const filePath = './data/hello.txt';

app.get('/', (req, res) => {
    const content = fs.existsSync(filePath) ? fs.readFileSync(filePath, 'utf-8') : "ফাইল এখনো খালি!";
    res.send(`<h1>Data from Volume:</h1><p>${content}</p>`);
});

app.post('/save', (req, res) => {
    const text = req.body.text;
    fs.appendFileSync(filePath, text + "\n");
    res.send("Data saved to volume!");
});

app.listen(3000, () => console.log("Server running on port 3000"));
```

```Dockerfile
FROM node:18
WORKDIR /app
RUN npm install express
COPY . .
RUN mkdir data
EXPOSE 3000
CMD ["node", "server.js"]
```

