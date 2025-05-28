---
layout: post
title: Using Docker for Cross Platform Development Building Containers for Linux Windows and macOS
subtitle: Streamline cross-platform development by building and running Docker containers on Linux, Windows, and macOS
categories: Docker
tags: [Docker, Cross-Platform, Containers, DevOps, Windows, Linux, macOS, Multi-Architecture]
excerpt: Learn how to use Docker to create and manage containers that work across Linux, Windows, and macOS. Explore techniques like multi-architecture builds, emulation, and best practices for cross-platform compatibility.
---
Modern development teams often need to support applications that run on multiple platforms — Linux, Windows, and macOS. Maintaining platform-specific environments manually is error-prone and inefficient.

**Docker** provides a consistent way to package and run applications across different operating systems using containers. In this post, we’ll explore how to use Docker for **cross-platform development**, covering key concepts like **multi-architecture images**, **buildx**, and **emulation** using QEMU.

---

#### Why Cross-Platform Containers?

Building cross-platform containers allows you to:

- Develop on **macOS or Windows**, but deploy to **Linux**
- Test platform-specific logic in isolation
- Package and distribute consistent artifacts across CI/CD
- Simplify dependency management across OS boundaries

---

#### Key Concepts

- **Base Images**: Containers start from base images that are often OS and architecture specific.
- **Image Architecture**: E.g., `linux/amd64`, `linux/arm64`, `windows/amd64`
- **Docker Buildx**: Advanced build tool for building multi-platform images
- **QEMU**: Emulates CPU architecture differences when building cross-architecture containers

---

#### Setting Up Docker for Cross-Platform Builds

1. **Enable Docker BuildKit** (already enabled in Docker Desktop):

```bash
export DOCKER_BUILDKIT=1
```

2. **Create a new buildx builder**:

```bash
docker buildx create --name multiarch-builder --use
docker buildx inspect --bootstrap
```

3. **Install QEMU** (for emulating different architectures):

On Docker Desktop (macOS/Windows), QEMU is preinstalled. On Linux:

```bash
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

---

#### Building Multi-Platform Images

Let’s build an image that supports `linux/amd64` and `linux/arm64`:

```bash
docker buildx build --platform linux/amd64,linux/arm64 \
-t myapp:multiarch \
--push \
.
```

- `--platform` specifies target architectures
- `--push` uploads the manifest to Docker Hub or registry

To build for **Windows**:

- Use `--platform windows/amd64`
- Ensure the base image (e.g., `mcr.microsoft.com/windows/servercore`) matches the host’s Windows version

```bash
docker buildx build --platform windows/amd64 -t myapp:win .
```

> ⚠️ Windows containers can only be built and run on **Windows hosts** with Windows containers enabled.

---

#### Example: Cross-Platform Dockerfile

```Dockerfile
FROM --platform=$BUILDPLATFORM node:18-alpine

WORKDIR /app
COPY . .

RUN npm install

CMD ["node", "index.js"]
```

Add platform awareness using `$TARGETPLATFORM` and `$BUILDPLATFORM`:

```Dockerfile
RUN echo "Building for $TARGETPLATFORM from $BUILDPLATFORM"
```

---

#### Running Containers on Different Platforms

- On **Linux/macOS**, containers are typically built for `linux/amd64`
- On **Apple Silicon (M1/M2)**, use `linux/arm64` or enable emulation for `amd64`
- On **Windows**, switch between Linux and Windows container engines via Docker Desktop

To test on a different architecture locally:

```bash
docker run --platform linux/arm64 myapp:multiarch
```

---

#### Testing with GitHub Actions (CI)

CI pipelines can automatically build multi-architecture images:

```yaml
jobs:
build:
runs-on: ubuntu-latest
steps:
- uses: actions/checkout@v2
- name: Set up QEMU
uses: docker/setup-qemu-action@v2
- name: Set up Docker Buildx
uses: docker/setup-buildx-action@v2
- name: Build and Push
uses: docker/build-push-action@v3
with:
platforms: linux/amd64,linux/arm64
push: true
tags: user/myapp:latest
```

---

#### Best Practices

- Use **multi-platform base images** (e.g., `node`, `python`, `openjdk`)
- Avoid OS-specific dependencies when writing cross-platform Dockerfiles
- Use **ARG TARGETPLATFORM** to adjust logic per platform
- Automate builds with **buildx** and **CI/CD pipelines**
- Test your image on each target platform before production release

---

#### Conclusion

Cross-platform development with Docker allows you to **build once and run anywhere** — whether that’s Linux, macOS, or Windows. By using tools like **Docker Buildx** and **QEMU**, you can easily build and test containers for multiple architectures, streamline development workflows, and ensure your applications are truly portable.

Mastering this setup means your software will be future-proof and ready for the diverse infrastructure of modern DevOps and cloud environments.
