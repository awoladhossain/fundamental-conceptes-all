# Docker Networks 🌐

## আগে বোঝো - Problem টা কী?
তুমি যদি দুইটা container চালাও:
```bash
Container 1: Node.js App (port 3000)
Container 2: MongoDB (port 27017)
```
- এরা কি নিজে নিজে কথা বলতে পারবে? না!
- কারণ প্রতিটা container হলো isolated — নিজের ভেতরে বন্দী।

```bash
[Node App Container]     [MongoDB Container]
      🔒                       🔒
   কথা বলতে পারছে না ❌
```
- Solution হলো Docker Network — এটা দিয়ে containers কে একই "নেটওয়ার্কে" রাখা যায়, তখন তারা নিজেদের সাথে কথা বলতে পারে।

## Real Life Analogy
```bash
Network ছাড়া:
তুমি আর তোমার বন্ধু দুইটা আলাদা বাড়িতে বন্দী
কোনো রাস্তা নেই → কথা হবে না

Network দিলে:
দুইটা বাড়ির মাঝে রাস্তা তৈরি হলো
এখন যোগাযোগ সম্ভব ✅
```
## Docker এ কতগুলো Network Type আছে?

```bash
1. bridge    → Default, most common (আজকে এটাই শিখবো)
2. host      → Container host এর network directly use করে
3. none      → কোনো network নেই, সম্পূর্ণ isolated
4. overlay   → Multiple machines এর মধ্যে (Swarm/K8s এ লাগে)
```
- আজকে bridge নিয়ে details করবো কারণ 90% কাজ এটা দিয়েই হয়।

## Bridge Network কীভাবে কাজ করে?

```bash
[Docker Host Machine]
┌─────────────────────────────────────┐
│                                     │
│   [bridge network: mynet]           │
│   ┌─────────┐     ┌─────────┐      │
│   │  app    │────▶│   db    │      │
│   │container│     │container│      │
│   └─────────┘     └─────────┘      │
│                                     │
└─────────────────────────────────────┘
```
### Same network এ থাকলে:
- Container গুলো নাম দিয়ে একে অপরকে চিনতে পারে
- db নামের container কে app container শুধু db লিখেই access করতে পারে
- IP address মনে রাখতে হয় না!

## Default Bridge vs Custom Bridge
Docker এ একটা default bridge network আছে, কিন্তু সেটা use করা উচিত না।

```bash
Default Bridge সমস্যা:
❌ Container নাম দিয়ে চেনা যায় না
❌ IP দিয়ে communicate করতে হয়
❌ IP change হলে সব ভাঙে

Custom Bridge:
✅ Container নাম দিয়ে চেনা যায় (DNS automatic)
✅ Isolated — শুধু same network এর containers কথা বলতে পারে
✅ Production ready
```

## Step by Step Practice

### Step 1: Default Network দেখো
```bash
# Docker এ কী কী network আছে দেখো
docker network ls
```

Output:
```
NETWORK ID     NAME      DRIVER    SCOPE
abc123         bridge    bridge    local
def456         host      host      local
ghi789         none      null      local
```
- এই `bridge` হলো default। এটাতে container গুলো IP দিয়ে কথা বলে, নাম দিয়ে না।

### Step 2: Default Bridge এর সমস্যা Prove করো

```bash
# দুইটা container চালাও, কোনো network specify করোনি
docker run -d --name container1 alpine sleep 3600
docker run -d --name container2 alpine sleep 3600
```
- `sleep 3600` মানে container টা 1 ঘন্টা চলতে থাকবে, কিছু করবে না — just alive থাকবে।

```bash
# container1 এর ভেতরে যাও
docker exec -it container1 sh
```
- Container এর ভেতরে:

```bash
# container2 কে নাম দিয়ে ping করো
ping container2
# Output: ping: bad address 'container2' ❌
# নাম দিয়ে চিনতে পারছে না!

exit
```

```bash
# cleanup
docker rm -f container1 container2
```
### Step 3: Custom Network তৈরি করো

```bash
# Custom bridge network তৈরি
docker network create mynet
```
Output:
```
a1b2c3d4e5f6...  (network এর ID)
```

```bash
# দেখো network তৈরি হয়েছে
docker network ls
```

Output:
```
NETWORK ID     NAME      DRIVER    SCOPE
abc123         bridge    bridge    local
def456         host      host      local
ghi789         none      null      local
xyz000         mynet     bridge    local  ✅
```

### Step 4: Custom Network এ DNS কাজ করে দেখো

```bash
# দুইটা container চালাও, এবার --network mynet দাও
docker run -d --name container1 --network mynet alpine sleep 3600
docker run -d --name container2 --network mynet alpine sleep 3600
```

```bash
# container1 এর ভেতরে যাও
docker exec -it container1 sh
```
- Container এর ভেতরে:

```bash
# container2 কে নাম দিয়ে ping করো
ping container2
# Output:
# PING container2 (172.18.0.3): 56 data bytes
# 64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.123 ms ✅
# কাজ করছে!

exit
```
- নাম দিয়ে চিনতে পারছে! এটাই Custom Network এর power।

```bash
# cleanup
docker rm -f container1 container2
```
### Step 5: Network Inspect করো

```bash
docker network inspect mynet
```
- Output (important অংশ):

```bash
[
    {
        "Name": "mynet",
        "Id": "01f3c05598fd017323583c8b8e41f3eb7a046bac46c821dfa74c496ae6b70d00",
        "Created": "2026-03-01T10:58:49.596105971Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.enable_ipv4": "true",
            "com.docker.network.enable_ipv6": "false"
        },
        "Labels": {}
    }
]
```
- Container চালালে `Containers` এর ভেতরে দেখা যাবে কে কে আছে।

### Step 6: Real World Example — Node App + MongoDB
এখন একটা **real scenario** practice করবো।

### ফাইল structure:

```bash
~/network-practice/
    index.js
    package.json
    Dockerfile
```

```bash
mkdir ~/network-practice
cd ~/network-practice
```
- package.json তৈরি করো:

```bash
{
  "name": "network-practice",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^7.0.0"
  }
}
```
- index.js তৈরি করো:

```bash
const express = require('express');
const mongoose = require('mongoose');

const app = express();
app.use(express.json());

// এখানে 'mongodb' হলো container এর নাম
// Docker DNS automatically resolve করবে
const MONGO_URL = 'mongodb://mongodb:27017/mydb';

mongoose.connect(MONGO_URL)
  .then(() => console.log('MongoDB connected ✅'))
  .catch(err => console.log('MongoDB error:', err));

// Simple Schema
const ItemSchema = new mongoose.Schema({ name: String });
const Item = mongoose.model('Item', ItemSchema);

// Routes
app.get('/', (req, res) => {
  res.json({ message: 'App চলছে!' });
});

app.post('/items', async (req, res) => {
  const item = await Item.create({ name: req.body.name });
  res.json(item);
});

app.get('/items', async (req, res) => {
  const items = await Item.find();
  res.json(items);
});

app.listen(3000, () => console.log('Server running on port 3000'));
```
- Dockerfile তৈরি করো:

```bash
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

- এখন Run করো:

### Step 1: Network তৈরি করো

```bash
docker network create appnet
```

### Step 2: MongoDB container চালাও

```bash
docker run -d \
  --name mongodb \
  --network appnet \
  -v mongodata:/data/db \
  mongo:6
```

- `--name mongodb` — এই নামটাই app container থেকে use হবে connection এ।

### Step 3: App এর Image build করো

```bash
docker build -t myapp .
```

### Step 4: App container চালাও

```bash
docker run -d \
  --name myapp \
  --network appnet \
  -p 3000:3000 \
  myapp
```

### Step 3: App এর Image build করো

```bash
docker build -t myapp .
```

### Step 4: App container চালাও

```bash
docker run -d \
  --name myapp \
  --network appnet \
  -p 3000:3000 \
  myapp
```
