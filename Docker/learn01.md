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

## Start and restart containers

```bash
docker start demo-nginx
docker ps
docker restart demo-nginx
docker ps
```

- `docker start` resumes a stopped container. `docker restart` stops and then starts a container in one command. Both preserve the container's data.

- The container returns to running state. Notice the container ID and name remain the same - it's the same container, not a new one.

1. Use 'start' for stopped containers, 'restart' for running ones
2. Restart is useful for applying configuration changes
3. The container retains its IP address and volumes after restart

## Pause and unpause containers

**Temporarily freeze a container's processes without stopping it.**

```bash
docker pause demo-nginx
docker ps
docker unpause demo-nginx
docker ps
```
- Pausing freezes all processes in the container using cgroups. The container remains in memory but consumes no CPU. Useful for temporarily halting resource-intensive containers.

- When paused, the container status shows 'Paused'. After unpause, it returns to 'Up' status and resumes immediately where it left off.

1. Paused containers still occupy memory
2. Use pause for debugging or temporarily reducing system load
3. Unlike stop/start, pause/unpause is instantaneous

## Execute commands in running containers

**Run commands inside a live container without stopping it.**

```bash
docker exec demo-nginx ls /usr/share/nginx/html
docker exec demo-nginx cat /etc/nginx/nginx.conf
docker exec -it demo-nginx bash
```

- `docker exec` runs a new process inside a running container. The `-it` flags give you an interactive terminal. Type 'exit' to leave without stopping the container.

- You'll see directory contents, config file content, and then get an interactive shell inside the running container.

1. Exec is perfect for debugging running containers
2. The container must be running to use exec
3. Common commands: bash, sh, cat, ls, ps
4. Type 'exit' to leave the shell - the container keeps running

## View container logs with filters

```bash
docker logs demo-nginx
docker logs --tail 20 demo-nginx
docker logs --since 5m demo-nginx
docker logs -f demo-nginx
```

- Logs show everything written to stdout/stderr. Use --tail to limit lines, --since for time range, and -f to follow in real-time. Press Ctrl+C to stop following.

- You'll see nginx access and error logs. The -f option streams new log entries as they appear.

1. Timestamps help correlate logs with events: --timestamps
2. Combine options: docker logs --tail 50 --since 1h container
3. Logs persist even after container stops
4. Use grep to filter: docker logs demo-nginx | grep error

## Detached Mode (-d)

- এটি সবচেয়ে বেশি ব্যবহৃত হয়। এর মানে হলো কন্টেইনারটি ব্যাকগ্রাউন্ডে চলবে। আপনি কমান্ড দেওয়ার পর কন্টেইনারটি চালু হয়ে যাবে এবং আপনি আপনার টার্মিনালটি অন্য কাজের জন্য ব্যবহার করতে পারবেন।

```bash
docker run -d -p 3000:3000 ts-docker:v1
```
- সুবিধা: টার্মিনাল বন্ধ করে দিলেও কন্টেইনার বন্ধ হবে না। এটি সার্ভার বা ডাটাবেজ রান করার জন্য পারফেক্ট।

```bash
docker start --attach ts-docker-container
```
1.  লাইভ লগ দেখা (Debugging): আপনার TypeScript অ্যাপ্লিকেশনে যদি কোনো বাগ (bug) থাকে বা কোড রান করার সময় কোনো এরর হয়, তবে --attach করলে সেই এরর মেসেজটি আপনি সরাসরি টার্মিনালে দেখতে পাবেন।

- উদাহরণ: ধরুন ডাটাবেজ কানেক্ট না হওয়ার কারণে অ্যাপটি ক্র্যাশ করছে। অ্যাটাচ করা থাকলে আপনি সাথে সাথেই Error: Connection refused লেখাটি দেখতে পাবেন।

2. ইন্টারঅ্যাক্টিভ কাজ (Input/Output): যদি আপনার অ্যাপটি ইউজারের কাছ থেকে কোনো ইনপুট চায়, তবে --attach ছাড়া আপনি সেই ইনপুট দিতে পারবেন না। এটি আপনার টার্মিনালকে কন্টেইনারের সাথে সরাসরি পাইপের মতো জুড়ে দেয়।

3. কন্টেইনারের বর্তমান অবস্থা বোঝা:
কখনও কখনও docker start দিলে মনে হয় কন্টেইনারটি চালু হয়েছে, কিন্তু আসলে সেটি ২ সেকেন্ড পরেই বন্ধ হয়ে যায়। --attach করলে আপনি দেখতে পাবেন কেন সেটি দ্রুত বন্ধ হয়ে যাচ্ছে (যেমন: কোনো ফাইল মিসিং বা কনফিগারেশন ভুল)।

## Inspect container details

```bash
docker inspect demo-nginx
docker inspect --format '{{.State.Status}}' demo-nginx
docker inspect --format '{{.NetworkSettings.IPAddress}}' demo-nginx
docker inspect --format '{{.Config.Image}}' demo-nginx
```
- Inspect returns detailed JSON about the container. Use --format with Go templates to extract specific fields like IP address, status, or volumes.

- First command shows full JSON. Subsequent commands show just the requested field: status, IP address, and image name.

1. Inspect shows everything: environment, volumes, networks, ports
2. Use jq for better JSON formatting: docker inspect demo-nginx | jq
3. Check State section for detailed status information
4. NetworkSettings shows IP, ports, and network configuration

## Monitor container resource usage
```bash
docker stats demo-nginx
docker stats --no-stream demo-nginx
```
- Stats displays live resource consumption. Without --no-stream, it updates continuously. With it, shows a single snapshot. Press Ctrl+C to exit continuous mode.

- You'll see a table with CPU %, memory usage, network I/O, and block I/O updating in real-time.

1. Run 'docker stats' (no name) to see all containers
2. Use this to identify resource-hungry containers
3. Memory % is of total system memory
4. NET I/O shows total data sent/received

## Copy files between host and container

```bash
echo '<h1>Hello from Docker!</h1>' > custom.html
docker cp custom.html demo-nginx:/usr/share/nginx/html/
docker exec demo-nginx cat /usr/share/nginx/html/custom.html
docker cp demo-nginx:/etc/nginx/nginx.conf ./nginx-backup.conf
```

- The `docker cp` command copies files between host and container. Works with both running and stopped containers. Format: source → destination.

- The HTML file is copied into the container and can be accessed. The config file is copied from container to your current directory.

1. Copy directories: docker cp ./folder container:/path/
2. Works even if container is stopped
3. Use this for quick config changes or log extraction
4. For persistent data, use volumes instead (covered in later lessons)

## Rename and update containers
```bash
docker rename demo-nginx my-webserver
docker ps
docker update --memory 512m my-webserver
docker inspect --format '{{.HostConfig.Memory}}' my-webserver
```
- Rename changes a container's name. Update modifies resource constraints like memory and CPU limits on running containers without restarting.

- The container now appears with the new name. Memory limit is set to 512MB (shown in bytes in inspect output).

1. Names must be unique across all containers
2. Update is useful for adjusting resources without downtime
3. Not all settings can be updated - some require recreating the container
4. Common updates: --memory, --cpu-shares, --restart

## View container differences
```bash
docker diff my-webserver
```
- Shows filesystem changes: A (added), D (deleted), C (changed). Useful for understanding what a container has modified.

- You'll see files added or changed, including the custom.html you copied earlier.


1. A = Added, C = Changed, D = Deleted
2. Helps debug containers with unexpected behavior
3. Shows why containers are larger than their images
4. Use this before committing a container to an image

## Attach to a running container
```bash
docker run -d --name test-ubuntu ubuntu bash -c 'while true; do date; sleep 2; done'
docker attach test-ubuntu
```
- Attach connects your terminal to the container's main process. Press Ctrl+C to detach (this will stop the container unless you use Ctrl+P, Ctrl+Q).

- You'll see date output every 2 seconds. Use Ctrl+P then Ctrl+Q to detach without stopping, or Ctrl+C to stop the container.

1. Attach is different from exec - it connects to PID 1
2. Use Ctrl+P, Ctrl+Q to detach without stopping
3. Exec is usually better for interactive work
4. Attach is useful for seeing real-time output

## Container cleanup strategies
```bash
docker stop my-webserver test-ubuntu
docker rm my-webserver test-ubuntu
docker ps -a
docker container prune
docker ps -a
```
- Stop containers before removing them, or use 'docker rm -f' to force remove. Prune removes all stopped containers at once - useful for bulk cleanup.

- Containers are removed from the system. Prune asks for confirmation and shows how much space was reclaimed.

1. Use 'docker rm -f $(docker ps -aq)' to remove all containers (careful!)
2. Add --rm flag when running containers for automatic cleanup
3. Prune regularly to avoid accumulating stopped containers
4. Remove containers before removing their images

## ভলিউম কী এবং কেন?

মনে করুন, আপনার একটা স্মার্টফোন আছে (Container)। ফোনের ভেতরে আপনি কিছু ছবি তুললেন। এখন ফোনটা যদি পানিতে পড়ে নষ্ট হয়ে যায় বা হারিয়ে যায়, তবে আপনার সব ছবিও শেষ।

কিন্তু আপনি যদি ফোনের ভেতরে একটা মেমোরি কার্ড বা এসডি কার্ড (Volume) লাগান এবং সব ছবি সেখানে সেভ করেন, তাহলে ফোন হারিয়ে গেলেও আপনার ছবি হারাবে না। আপনি ওই এসডি কার্ডটা খুলে অন্য যেকোনো ফোনে লাগালেই সব ছবি ফেরত পাবেন।

``ডকার ভলিউম ঠিক ওই এসডি কার্ডের মতো কাজ করে।``

1. কন্টেইনারের সাথে এর কাহিনী কী?
ডকার কন্টেইনারের ভেতরে আপনি যা-ই করবেন (কোনো ফাইল সেভ করা, ডাটাবেজ বানানো), তা কন্টেইনার ডিলিট করার সাথে সাথে মুছে যায়।

- ভলিউম ছাড়া: কন্টেইনার ডিলিট = সব ডাটা ডিলিট। ❌

- ভলিউম সহ: কন্টেইনার ডিলিট = ডাটা হার্ডড্রাইভে (ভলিউমে) সুরক্ষিত। ✅

## 🛠️ হাতে কলমে প্রথম পাঠ (ভলিউম তৈরি ও ব্যবহার)
চলুন আমরা একটা এসডি কার্ড (Volume) কিনি এবং সেটা কন্টেইনারে লাগাই।
