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

