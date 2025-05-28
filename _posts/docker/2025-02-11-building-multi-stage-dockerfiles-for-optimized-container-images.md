---
layout: post
title: Building Multi Stage Dockerfiles for Optimized Container Images
subtitle: Learn how to use multi-stage builds in Docker to create smaller, secure, and production-ready images
categories: Docker
tags: [Docker, Containers, DevOps, CI/CD, Dockerfile, Optimization, Container Security]
excerpt: Discover how to use multi-stage Dockerfiles to reduce image size, improve build performance, and separate build and runtime environments. Includes practical examples and best practices for production.
---
Docker simplifies application packaging and deployment, but naïve Dockerfiles often produce **bloated and insecure images**. The solution? **Multi-stage Docker builds** — a technique that allows you to separate the build and runtime environments to create **optimized, production-ready containers**.

This post explores how to write multi-stage Dockerfiles, why they're essential for clean builds, and how they help enforce best practices around **image size**, **security**, and **layer caching**.

---

#### What Are Multi-Stage Dockerfiles?

A **multi-stage Dockerfile** uses multiple `FROM` instructions to define separate stages in the build process.

Each stage:
- Has its own base image
- Can copy files from previous stages
- Is discarded unless explicitly copied, minimizing the final image size

This pattern lets you **build artifacts in one stage** and **copy only what's needed** into a final minimal image.

---

#### Benefits of Multi-Stage Builds

- 🧹 **Smaller Images** – Exclude compilers, build tools, and temp files
- 🔐 **Improved Security** – Avoid unnecessary binaries in production
- 🏎️ **Better Caching** – Separate build dependencies for faster rebuilds
- 🧪 **Simplified CI/CD** – One Dockerfile for dev, test, and prod

---

#### Example: Node.js Application

A simple Node.js app built with a multi-stage Dockerfile:

```dockerfile
# Stage 1: Build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Runtime
FROM node:18-slim
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/package*.json ./
RUN npm install --only=production
CMD ["node", "dist/index.js"]
```

✅ Result: Only the compiled output and runtime dependencies are in the final image.

---

#### Example: Go Application (Statically Compiled)

Go makes multi-stage builds especially efficient:

```dockerfile
# Stage 1: Build
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o app

# Stage 2: Final minimal image
FROM alpine:3.18
WORKDIR /app
COPY --from=builder /app/app .
ENTRYPOINT ["./app"]
```

✅ Final image is ~10MB vs 800MB+ in a single-stage build.

---

#### Best Practices for Multi-Stage Dockerfiles

1. **Use specific tags**  
   Avoid `latest`. Pin to `node:18`, `golang:1.21`, etc.

2. **Minimize layers in final stage**  
   Merge `COPY` and `RUN` commands where possible.

3. **Add `.dockerignore`**  
   Prevent unnecessary files from entering the build context.

4. **Use Alpine-based images**  
   For minimal base layers when possible.

5. **Use build arguments (ARG)**  
   Parameterize version numbers or envs.

```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}
```

---

#### Multi-Stage for Testing

Add a testing stage before production:

```dockerfile
# Stage 1: Test
FROM node:18 AS test
WORKDIR /app
COPY . .
RUN npm ci && npm test

# Stage 2: Build
FROM node:18 AS build
COPY . .
RUN npm run build

# Stage 3: Production
FROM node:18-slim
COPY --from=build /app/dist ./dist
CMD ["node", "dist/index.js"]
```

Only proceed to final image if tests pass ✅

---

#### Bonus: Image Size Comparison

| Approach             | Image Size | Layers |
|----------------------|------------|--------|
| Single-stage         | ~800MB     | Many   |
| Multi-stage (Node.js)| ~180MB     | Few    |
| Multi-stage (Go)     | ~10MB      | Minimal|

---

#### Conclusion

**Multi-stage Docker builds** are an essential technique for creating lean, secure, and production-ready container images. By separating build and runtime concerns, you not only reduce image size but also simplify security and CI/CD practices.

Whether you're building with **Node.js**, **Go**, **Java**, or **Python**, multi-stage Dockerfiles help ensure your containers are **efficient, maintainable, and deployable at scale**.
