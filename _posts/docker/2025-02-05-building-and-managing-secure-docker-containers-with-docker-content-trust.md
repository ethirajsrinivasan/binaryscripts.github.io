---
layout: post
title: Building and Managing Secure Docker Containers with Docker Content Trust
subtitle: Use Docker Content Trust to ensure container image integrity and prevent supply chain attacks
categories: Docker
tags: [Docker, Container Security, Docker Content Trust, Notary, DevSecOps, Image Signing, Secure CI/CD]
excerpt: Learn how to secure your container pipelines using Docker Content Trust. Explore best practices for signing and verifying Docker images to prevent tampering and enhance supply chain security.
---
As containers become the backbone of modern software delivery, securing them is more important than ever. One of the most critical aspects of container security is ensuring the **authenticity and integrity** of Docker images throughout the CI/CD pipeline.

**Docker Content Trust (DCT)** provides a mechanism for **cryptographically signing container images** and verifying those signatures at runtime. It protects against **tampered, unverified, or malicious images** entering your infrastructure.

In this blog, we’ll walk through how to **enable, use, and manage Docker Content Trust**, and integrate it into your **DevSecOps workflow** for building and managing secure containers.

---

#### What is Docker Content Trust?

**Docker Content Trust** leverages **The Update Framework (TUF)** and **Notary** to enable:

- **Signing Docker images**
- **Verifying image signatures** before pull or push
- **Preventing unsigned or tampered images** from entering your pipeline

With DCT enabled:
- Only signed images can be pushed to or pulled from a registry.
- Verification of publisher identity is enforced.

---

#### Enabling Docker Content Trust

To enable DCT in your terminal or build environment:

```bash
export DOCKER_CONTENT_TRUST=1
```

With DCT enabled:
- `docker push` requires image signing
- `docker pull` requires image verification
- `docker run` enforces verification before launch

To disable temporarily:

```bash
export DOCKER_CONTENT_TRUST=0
```

---

#### Signing Docker Images

To sign images, you must first have a **Notary key pair**.

Docker generates and stores:
- **Root key** (locally, never leaves your system)
- **Repository key** (used to sign tags)

Steps to sign and push a Docker image:

```bash
# Step 1: Build the image
docker build -t myregistry.com/app:v1 .

# Step 2: Log in to your Docker registry
docker login myregistry.com

# Step 3: Push with signing (DCT must be enabled)
DOCKER_CONTENT_TRUST=1 docker push myregistry.com/app:v1
```

The image will be signed and stored in the registry along with the signature metadata.

---

#### Verifying Docker Images

To verify an image signature:

```bash
DOCKER_CONTENT_TRUST=1 docker pull myregistry.com/app:v1
```

If the image has not been signed, or if the signature is invalid or missing, the pull will fail.

---

#### Key Management and Delegation

Docker Content Trust uses:

- **Root keys** – held securely and rotated rarely
- **Repository keys** – used for signing tags
- **Delegation keys** – grant access to other teams/services

To rotate keys:

1. Revoke old keys via Notary
2. Create new keys
3. Delegate signing roles

Best practice:
- Use a **hardware security module (HSM)** or **YubiKey** to store root keys
- Use **delegation keys** for CI/CD pipelines, not the root key

---

#### Protecting Against Supply Chain Attacks

DCT helps prevent:

- **Tampered images** in registries
- **Man-in-the-middle attacks** during image download
- **Unverified third-party images** in build environments

Combined with tools like **Trivy**, **Clair**, or **Docker Scout**, DCT forms a core part of your **container supply chain security posture**.

---

#### Integrating DCT in CI/CD Pipelines

In GitLab CI, GitHub Actions, or Jenkins, enable DCT before the build stage:

```yml
# GitLab CI example
stages:
- build
- deploy

build_image:
stage: build
script:
- export DOCKER_CONTENT_TRUST=1
- docker build -t myregistry.com/app:v2 .
- docker push myregistry.com/app:v2
```

Ensure that your CI runners have access to the **repository signing keys**.

---

#### Limitations of Docker Content Trust

- Only works with **Docker Hub** and **Notary v1**
- Not supported by all registries (e.g., ECR support is limited)
- Does not scan for image vulnerabilities (use a separate scanner)
- Signature metadata is not visible in `docker images` CLI

For broader adoption, consider switching to **Notary v2** (OCI signing standard) as it matures.

---

#### Best Practices

- Always enable DCT in production and staging environments
- Rotate and back up root keys securely
- Delegate signing to CI/CD service accounts
- Use `DOCKER_CONTENT_TRUST=1` in container build scripts
- Use image scanning tools for runtime vulnerability detection
- Prefer **immutable tags** (avoid re-tagging signed images)

---

#### Conclusion

**Docker Content Trust** is a powerful and easy-to-use tool to secure your container pipeline by **verifying image authenticity and integrity**. When combined with **good key management**, **CI/CD integration**, and **runtime scanning**, DCT helps fortify your containerized infrastructure against a wide range of supply chain attacks.

Start by signing your critical application images today — and build trust into every container you deploy.
