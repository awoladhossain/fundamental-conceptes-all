## Core Concept & Architecture (ডকারের)
সহজ কথায়, ডকার হলো OS-level virtualization। এটি হাইপারভাইজরের মতো পুরো হার্ডওয়্যার সিমুলেট করে না, বরং হোস্ট অপারেটিং সিস্টেমের Kernel শেয়ার করে।

**ডকারের আসল জাদুকর:**
ডকার মূলত লিনাক্স কার্নেলের দুটি প্রধান ফিচারের ওপর দাঁড়িয়ে:

- Namespaces: এটি প্রসেসগুলোকে একে অপরের থেকে আলাদা (Isolate) রাখে। যেমন: Network, PID, Mount namespaces। এর ফলে একটি কন্টেইনার মনে করে সে পুরো সিস্টেমে একাই আছে।
- Control Groups (cgroups): এটি রিসোর্স লিমিট সেট করে। অর্থাৎ, কোনো কন্টেইনার কতটা CPU বা RAM ব্যবহার করতে পারবে, তা এটি নিয়ন্ত্রণ করে।

**ডকার ইমেজ বনাম কন্টেইনার:**

- Image: এটি একটি Read-only টেমপ্লেট, যা অনেকগুলো Layers দিয়ে তৈরি (Union File System)।
- Container: এটি ইমেজের একটি রানটাইম ইন্সট্যান্স। ইমেজের ওপর একটি পাতলা Writable Layer যোগ করলেই সেটি কন্টেইনার হয়ে যায়।


## ডকার কখন 'ফেল' করে:

- Kernel Panic: কোনো কন্টেইনার যদি হোস্টের কার্নেলে এমন কোনো রিকোয়েস্ট পাঠায় যা কার্নেল হ্যান্ডেল করতে পারছে না, তবে পুরো সার্ভার ক্র্যাশ করতে পারে।
- Zombie Processes: কন্টেইনারের ভেতরে অনেক সময় PID 1 হিসেবে ঠিকমতো প্রসেস হ্যান্ডেল না করলে অপ্রয়োজনীয় প্রসেস জমে মেমোরি শেষ করে ফেলে।
- Disk Space Exhaustion: ডকারের লেয়ারিং সিস্টেমের কারণে অনেক সময় 'Dangling Images' বা অব্যবহৃত ভলিউম ডিস্ক ফুল করে দেয়।

## ইমেজ ম্যানেজমেন্ট (ইমেজ নিয়ে কাজ করার জন্য)
ইমেজ হলো আপনার অ্যাপ্লিকেশনের "ব্লু-প্রিন্ট"।

1. `docker build -t <image-name> . `: আপনার Dockerfile থেকে একটি ইমেজ তৈরি (Build) করে। -t দিয়ে ইমেজের একটা নাম দেওয়া হয়।

2. `docker images` : আপনার কম্পিউটারে বর্তমানে কতগুলো ইমেজ ডাউনলোড করা বা তৈরি করা আছে তার লিস্ট দেখায়।

3. `docker rmi <image-id>` : অপ্রয়োজনীয় কোনো ইমেজ ডিলিট করার জন্য। (rmi = Remove Image)

4. `docker pull <image-name>` : Docker Hub থেকে কোনো ইমেজ ডাউনলোড করার জন্য (যেমন: docker pull postgres)।

## কন্টেইনার লাইফসাইকেল (কন্টেইনার চালানো ও বন্ধ করা)

1. `docker run <image-name>` : ইমেজ থেকে একটি নতুন কন্টেইনার তৈরি করে এবং সেটি চালু করে।

2. `docker run -d -p 5000:5000 <image-name>` :
 -  `-d (Detached mode)`: কন্টেইনার ব্যাকগ্রাউন্ডে চলবে।
 -  `-p (Port mapping)`: হোস্টের ৫০০০ পোর্টের সাথে কন্টেইনারের ৫০০০ পোর্ট কানেক্ট করবে।

3. `-d (Detached mode)`: কন্টেইনার ব্যাকগ্রাউন্ডে চলবে।

4. `-p (Port mapping)`: হোস্টের ৫০০০ পোর্টের সাথে কন্টেইনারের ৫০০০ পোর্ট কানেক্ট করবে।

5. `docker ps` : বর্তমানে কোন কোন কন্টেইনার চলছে তার লিস্ট দেখায়।

6. `docker ps -a` : সব কন্টেইনারের লিস্ট দেখায় (সেটা চলুক বা বন্ধ থাকুক)।

7. `docker stop <container-id>` : রানিং কোনো কন্টেইনারকে থামিয়ে দেয়।

8. `docker start <container-id>` : বন্ধ হয়ে থাকা (Exited) কোনো কন্টেইনারকে আবার চালু করে।

9. `docker rm <container-id>` : কোনো কন্টেইনার পার্মানেন্টলি ডিলিট করে দেয়।


## কন্টেইনারের ভেতরে উঁকি দেওয়া (Debugging)
ডেভেলপমেন্টের সময় কন্টেইনারের ভেতর কী হচ্ছে তা দেখার জন্য এগুলো লাগে।

1. `docker logs <container-id>` : কন্টেইনারের কনসোলে কী কী আউটপুট বা এরর আসছে তা দেখার জন্য।

2. `docker logs -f <container-id>` : লাইভ লগ দেখার জন্য (যেমনটা আপনি টার্মিনালে দেখেন)।

3. `docker exec -it <container-id> sh (বা bash)` : কন্টেইনারের ভেতরে ঢোকার জন্য। অনেকটা SSH করার মতো। এর ভেতরে ঢুকে আপনি ফাইল চেক করতে পারেন।

4. `docker inspect <container-id>` : কন্টেইনারের IP address, মাউন্ট করা ভলিউমসহ সব টেকনিক্যাল ডিটেইলস দেখার জন্য।

## হাউস-কিপিং
ডকার অনেক সময় প্রচুর ডিস্ক স্পেস দখল করে ফেলে।

1. `docker system prune` : এক কমান্ডে সব বন্ধ থাকা কন্টেইনার, অব্যবহৃত নেটওয়ার্ক এবং ক্যাশ ডিলিট করে দেয়। (সাবধানে ব্যবহার করবেন!)

2. `docker volume ls` : আপনার সিস্টেমে কতগুলো ডকার ভলিউম (যেখানে ডাটা সেভ থাকে) আছে তা দেখার জন্য।

## Run a web server container

```bash
docker run -d -p 8080:80 --name my-nginx nginx
```

- The `-d` flag runs in detached mode (background). `-p 8080:80` maps port 8080 on your host to port 80 in the container. `--name` gives it a friendly name instead of a random one.

- You should see a long container ID printed, and your terminal prompt returns immediately. The container runs in the background.

1. Open http://localhost:8080 in your browser to see nginx running!
2. The `-d` flag is crucial for long-running services
3. Port mapping format is always `host:container`

## View running containers and logs
```bash
docker ps
```
```bash
docker logs my-nginx
```
- `docker ps` (without -a) shows only running containers. `docker logs` displays what the container has output to stdout/stderr.
- You should see `my-nginx` listed as running (UP status) and nginx access logs when you view the logs.

1. Refresh http://localhost:8080 in your browser and run `docker logs my-nginx` again to see new log entries
2. Use `docker logs -f my-nginx` to follow logs in real-time (like `tail -f`)

## Stop and remove containers
```bash
docker stop my-nginx
docker ps -a
docker rm my-nginx
docker ps -a
```

- `docker stop` gracefully stops a running container. `docker rm` removes a stopped container completely. Notice how the container disappears from the list after removal.
- After `stop`, the container status changes to "Exited". After `rm`, it's completely gone from `docker ps -a`.

1. You can combine: `docker rm -f my-nginx` to force-remove a running container
2. Use `docker container prune` to remove all stopped containers at once
3. Removing a container doesn't remove the image - it's still cached

## Practice: Run and interact with a container

```bash
docker run -d -p 8081:80 --name my-apache httpd
docker ps
docker logs my-apache
docker stop my-apache
docker rm my-apache
```
- This follows the full container lifecycle: run in background, check it's running, view logs, stop it, and clean up. The `httpd` image is Apache web server.

- You should be able to visit http://localhost:8081 and see Apache's default page. Then successfully stop and remove the container.

1. Try different port numbers to avoid conflicts
2. Practice naming containers with `--name` for easier management
3. Use `docker images` to see all the images you've downloaded


## Search for images on Docker Hub
```bash
docker search nginx
docker search --filter stars=1000 nginx
```

- The `docker search` command queries Docker Hub for images. The filter option helps you find popular, well-maintained images with many stars.

- You'll see a list of nginx-related images with columns for NAME, DESCRIPTION, STARS, OFFICIAL, and AUTOMATED.

1. Look for OFFICIAL images (marked [OK]) - they're maintained by Docker or the software vendor
2. High star count usually indicates a well-maintained, popular image
3. Check the image description for specific use cases

## Pull a specific image

```bash
docker pull nginx
```
- This downloads the nginx image from Docker Hub. Without a tag, it defaults to 'latest'. The image is stored locally and can be used to create containers.

- You'll see download progress bars for each layer. Output ends with something like 'Status: Downloaded newer image for nginx:latest'.

1. Images are downloaded in layers - Docker reuses layers across images to save space
2. The first pull takes time, but subsequent pulls reuse existing layers

## Understanding image tags

```bash
docker pull nginx:1.25
docker pull nginx:alpine
docker images nginx
```
- Tags specify versions or variants. Format is `image:tag`. Alpine variants are minimal Linux distributions, resulting in smaller images.

- You'll see three nginx images listed with different tags: latest, 1.25, and alpine. Notice the size differences.

1. Alpine images are typically 5-10x smaller than standard images

2. Use specific version tags (like 1.25) in production to ensure consistency

3. Check Docker Hub for available tags and their documentation

## Inspect image details

```bash
docker inspect nginx:latest
docker history nginx:latest
```
- `docker inspect` shows JSON metadata including config, layers, and environment. `docker history` displays the build commands that created each layer.

- You'll see detailed JSON output from inspect, and a table of layers with their sizes from history.

1. Use `docker inspect nginx:latest | grep -A 10 Env` to see just environment variables
2. History shows how the image was built - useful for learning Dockerfile patterns
3. Layers marked <missing> are from multi-stage builds or external registries

## List all local images

```bash
docker images
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```
- The first command shows all images. The second formats output to show only repository, tag, and size for clarity.
- You'll see a table with all your images including nginx variants, ubuntu, hello-world, and possibly httpd from Lesson 101.

1. Images are identified by REPOSITORY:TAG or IMAGE ID
2. The same image can have multiple tags (aliases)
3. Size shown is the compressed size; actual disk usage may differ due to shared layers

## Understanding image layers

```bash
docker pull python:3.11
docker pull python:3.11-alpine
docker images python
```

- Docker images are built in layers. Each instruction in a Dockerfile creates a layer. Layers are cached and reused across images.
- Notice the size difference between python:3.11 (~900MB) and python:3.11-alpine (~50MB). Both work but alpine is minimal.

1. Alpine images save disk space and reduce attack surface
2. Standard images include more tools, useful for debugging
3. Choose alpine for production, standard for development

## Tag an existing image
**Create your own tag (alias) for an image.**

```bash
docker tag nginx:latest my-nginx:v1
docker images | grep nginx
```
- Tagging creates a new name pointing to the same image. Useful for versioning or pushing to private registries.
- You'll see both 'nginx:latest' and 'my-nginx:v1' listed with the same IMAGE ID.

1. Tags are just labels - multiple tags can point to the same image
2. Use meaningful tag names for your custom images
3. Format: `repository:tag` or `registry/repository:tag` for remote registries

## View image disk usage
```bash
docker system df
docker system df -v
```
- This shows storage used by images, containers, and volumes. The -v flag provides detailed information per image.
- You'll see a summary showing IMAGES, CONTAINERS, and VOLUMES with their sizes and reclaimable space.

1. Shared layers mean total size is less than sum of all images
2. Reclaimable space shows what can be freed by removing unused images
3. Run this regularly to monitor Docker's disk usage

## Remove unused images

```bash
docker rmi python:3.11
docker images
docker image prune -a
```

- `docker rmi` removes a specific image. `docker image prune -a` removes all unused images (not referenced by any container).
- The python:3.11 image is removed. Prune shows how much space was reclaimed.

1. Can't remove images used by containers (even stopped ones)
2. Remove containers first with `docker container prune`
3. The -a flag is important - without it, only dangling images are removed

## Practice: Find and pull a specific image version

**Apply what you've learned to work with a different image.**

```bash
docker search redis
docker pull redis:7-alpine
docker inspect redis:7-alpine | grep -A 5 ExposedPorts
docker rmi redis:7-alpine
```

- This workflow demonstrates searching, pulling a specific tagged version, inspecting configuration, and cleaning up.
- You successfully pull redis 7 alpine variant, see it exposes port 6379, then remove it.

1. Always check the official image documentation on Docker Hub
2. Use alpine variants when image size matters
3. Inspect images to understand their configuration before running


## Understanding container states

**Learn about the different states a container can be in: created, running, paused, exited.**

```bash
docker run -d --name demo-nginx nginx
docker ps
docker stop demo-nginx
docker ps -a
```

- Containers transition through states: created → running → stopped (exited). The `ps` command shows running containers, while `ps -a` shows all containers regardless of state.

- You'll see the container appear in 'docker ps' with UP status, then after stopping it will show 'Exited' status in 'docker ps -a'.

1. Running containers consume resources; stopped containers don't
2. Stopped containers retain their filesystem and configuration
3. You can restart stopped containers without losing data

