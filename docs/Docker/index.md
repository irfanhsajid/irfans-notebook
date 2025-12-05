# Docker

Docker is a platform for developing, shipping, and running applications in containers. Containers package an application with all its dependencies, making it portable and consistent across different environments.

## Installation

### Linux

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group (to run without sudo)
sudo usermod -aG docker $USER
```

### macOS/Windows

Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop)

## Basic Concepts

### Images

Read-only templates used to create containers. Think of them as application blueprints.

### Containers

Running instances of images. Isolated environments where your application runs.

### Dockerfile

A text file containing instructions to build a Docker image.

### Docker Compose

A tool for defining and running multi-container applications.

## Basic Commands

### Images

```bash
# List images
docker images

# Pull an image
docker pull nginx:latest

# Build an image
docker build -t myapp:1.0 .

# Remove an image
docker rmi nginx:latest

# Remove unused images
docker image prune
```

### Containers

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Run a container
docker run nginx

# Run container in background (detached mode)
docker run -d nginx

# Run with port mapping
docker run -d -p 8080:80 nginx

# Run with name
docker run -d --name my-nginx nginx

# Stop a container
docker stop my-nginx

# Start a stopped container
docker start my-nginx

# Restart a container
docker restart my-nginx

# Remove a container
docker rm my-nginx

# Remove all stopped containers
docker container prune
```

### Logs and Inspection

```bash
# View container logs
docker logs my-nginx

# Follow logs (live tail)
docker logs -f my-nginx

# Inspect container
docker inspect my-nginx

# View container stats
docker stats my-nginx

# Execute command in running container
docker exec -it my-nginx bash
```

## Dockerfile

### Basic Dockerfile

```dockerfile
# Use base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Start application
CMD ["npm", "start"]
```

### Multi-stage Build

```dockerfile
# Build stage
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY package*.json ./
RUN npm install --production
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Best Practices

```dockerfile
FROM node:18-alpine

# Use specific versions
WORKDIR /app

# Copy package files first (better caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --production

# Copy source code
COPY . .

# Use non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3000

CMD ["node", "index.js"]
```

## Docker Compose

### docker-compose.yml

```yaml
version: "3.8"

services:
  # Web application
  app:
    build: .
    container_name: myapp
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped

  # PostgreSQL database
  db:
    image: postgres:15-alpine
    container_name: postgres
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  # Redis cache
  redis:
    image: redis:7-alpine
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

### Docker Compose Commands

```bash
# Start services
docker-compose up

# Start in background
docker-compose up -d

# Build and start
docker-compose up --build

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f app

# Restart service
docker-compose restart app

# Execute command in service
docker-compose exec app sh
```

## Common Use Cases

### Node.js Application

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --production

COPY . .

EXPOSE 3000

CMD ["node", "index.js"]
```

```yaml
version: "3.8"

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - .:/app
      - /app/node_modules
```

### React Development

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

```yaml
version: "3.8"

services:
  react-app:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - CHOKIDAR_USEPOLLING=true
```

### Next.js Application

```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/next.config.js ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json

EXPOSE 3000
CMD ["npm", "start"]
```

### Full Stack with Database

```yaml
version: "3.8"

services:
  # Next.js app
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:4000
    depends_on:
      - backend

  # Node.js API
  backend:
    build: ./backend
    ports:
      - "4000:4000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  # PostgreSQL
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Redis
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## Networking

### Create Network

```bash
docker network create my-network
```

### Connect Container to Network

```bash
docker run -d --name app --network my-network nginx
```

### Docker Compose Networking

```yaml
version: "3.8"

services:
  app:
    image: nginx
    networks:
      - frontend
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
  backend:
```

## Volumes

### Named Volumes

```bash
# Create volume
docker volume create my-data

# Use volume
docker run -v my-data:/app/data nginx

# List volumes
docker volume ls

# Remove volume
docker volume rm my-data
```

### Bind Mounts

```bash
# Mount local directory
docker run -v $(pwd):/app nginx

# Read-only mount
docker run -v $(pwd):/app:ro nginx
```

## Environment Variables

### Pass Environment Variables

```bash
# Single variable
docker run -e NODE_ENV=production nginx

# Multiple variables
docker run -e NODE_ENV=production -e PORT=3000 nginx

# From file
docker run --env-file .env nginx
```

### .env File

```bash
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://localhost/mydb
```

## Useful Commands

### Copy Files

```bash
# Copy from host to container
docker cp ./file.txt my-container:/app/

# Copy from container to host
docker cp my-container:/app/file.txt ./
```

### Cleanup

```bash
# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune -a

# Remove all unused volumes
docker volume prune

# Remove all unused networks
docker network prune

# Remove everything unused
docker system prune -a --volumes
```

### Resource Management

```bash
# Limit CPU and memory
docker run -d --cpus="1.5" --memory="512m" nginx

# View resource usage
docker stats
```

## .dockerignore

```
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
.DS_Store
*.md
dist
build
.next
```

## Best Practices

1. **Use official base images** when possible
2. **Use specific image versions** (not `latest`)
3. **Minimize layers** by combining RUN commands
4. **Use .dockerignore** to exclude unnecessary files
5. **Don't run as root** - create a user
6. **Use multi-stage builds** for smaller production images
7. **Cache dependencies** separately from source code
8. **Use environment variables** for configuration
9. **Health checks** to monitor container health
10. **Keep images small** - use alpine variants

## Health Checks

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY . .

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node healthcheck.js || exit 1

CMD ["node", "index.js"]
```

```yaml
version: "3.8"

services:
  app:
    build: .
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
```
