# 🐳 Docker Learning Notes

## 📂 Topics Covered

| File | Description | Status |
|------|-------------|--------|
| [Learn 01](learn01.md) | Docker Basics & Introduction | ✅ Done |
| [Network](network.md) | Docker Networking | ✅ Done |
| [Volume](volume.md) | Docker Volumes & Storage | ✅ Done |

## 🎯 Quick Reference

### Basic Commands
```bash
# Pull image
docker pull nginx

# Run container
docker run -d -p 80:80 nginx

# List containers
docker ps -a

# Stop container
docker stop <container_id>

# Remove container
docker rm <container_id>

# List images
docker images

# Remove image
docker rmi <image_id>
```

### Docker Compose
```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "80:80"
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: secret
```

### Dockerfile Example
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### Network Types
- **bridge** - Default, isolated network
- **host** - Uses host's network
- **none** - No networking
- **overlay** - Multi-host networking

## 📚 Resources
- [Docker Official Docs](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)

---
*Last Updated: March 2026*
