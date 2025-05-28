---
layout: post
title: Managing Secrets in Docker Using Docker Secrets and HashiCorp Vault
subtitle: Securely manage sensitive credentials in Docker using built-in secrets management and external tools like Vault
categories: Docker
tags: [Docker, DevSecOps, Secrets Management, Vault, HashiCorp, Docker Swarm, Security, Containers]
excerpt: Learn how to securely manage secrets like API keys and passwords in Docker using Docker Secrets and HashiCorp Vault. Explore best practices for container security and production-ready deployments.
---
Managing secrets such as **API keys**, **database passwords**, and **TLS certificates** securely is a critical part of building secure containerized applications. Exposing credentials via environment variables or configuration files can lead to security breaches.

Docker provides a built-in **secrets management system**, and tools like **HashiCorp Vault** offer dynamic and centralized secret storage with fine-grained access control.

In this blog, we’ll explore two secure approaches to secrets management in Docker:
1. Using **Docker Secrets** (native to Docker Swarm)
2. Integrating **HashiCorp Vault** for dynamic secret retrieval

---

#### Why Not Use Environment Variables?

While it's common to pass secrets using environment variables (`ENV DB_PASSWORD=secret123`), this method is insecure because:
- Env vars are visible via `docker inspect`
- They can leak into logs or be inherited by child processes
- There’s no access control or audit logging

For production, always use a **dedicated secrets management system**.

---

#### Method 1: Using Docker Secrets (Docker Swarm)

**Docker Secrets** is built into Docker Swarm and provides a secure way to manage secrets like passwords, tokens, or certs.

##### Step 1: Create a Secret

```bash
echo "my-db-password" | docker secret create db_password -
```

##### Step 2: Deploy a Service with the Secret

```bash
docker service create \
--name my-app \
--secret db_password \
myapp:latest
```

Inside the container, the secret is available as a **read-only file** at:

```
/run/secrets/db_password
```

##### Step 3: Access the Secret in Code

Example in Python:

```python
with open('/run/secrets/db_password') as f:
db_password = f.read().strip()
```

**Security Advantages:**
- Secrets are encrypted at rest
- Only accessible by the container
- Never exposed via environment or image layers

---

#### Method 2: Using HashiCorp Vault with Docker

**HashiCorp Vault** is a powerful tool for managing dynamic secrets across environments, supporting:
- Token-based or AppRole authentication
- Access policies
- Audit logging
- Secret rotation

##### Step 1: Run Vault (dev mode for testing)

```bash
docker run --cap-add=IPC_LOCK -d --name vault \
-p 8200:8200 \
-e 'VAULT_DEV_ROOT_TOKEN_ID=myroot' \
-e 'VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200' \
hashicorp/vault
```

##### Step 2: Store a Secret

```bash
export VAULT_ADDR=http://localhost:8200
export VAULT_TOKEN=myroot

vault kv put secret/db password=my-db-password
```

##### Step 3: Docker App Fetches Secret

Use Vault’s API or SDK to access secrets in your app:

```python
import hvac

client = hvac.Client(
url='http://vault:8200',
token='myroot'
)

response = client.secrets.kv.v2.read_secret_version(path='db')
db_password = response['data']['data']['password']
```

**Security Best Practices:**
- Never hardcode the root token in production
- Use **AppRole**, **JWT**, or **Kubernetes Auth** for dynamic identity-based auth
- Enable **audit logging** and **TTL-based secret expiry**
- Use **Vault Agent** or **Sidecar Injection** to cache and renew tokens

---

#### Docker + Vault Sidecar Pattern (Production)

Instead of embedding Vault logic into your app, run a sidecar:

- Vault Agent runs as a sidecar container
- Fetches secrets from Vault
- Renders them to shared volume as files
- Your app reads from the volume

```yaml
volumes:
- ./vault-agent-config:/vault/config
- shared-secret:/mnt/secrets

services:
vault-agent:
image: vault
command: agent -config=/vault/config/agent.hcl

app:
image: myapp:latest
volumes:
- shared-secret:/mnt/secrets
```

This decouples secret fetching from application logic and improves maintainability.

---

#### Best Practices for Secrets Management in Docker

- **Never bake secrets into Docker images**
- Avoid using **environment variables** for sensitive data
- Use **Docker Secrets** for Swarm-managed apps
- Use **Vault** for dynamic, cloud-native secrets management
- Rotate secrets regularly and audit access
- Secure API endpoints used to fetch secrets
- Prefer **file mounts** and read-only secrets

---

#### Conclusion

Secrets management is a vital part of container security. With **Docker Secrets**, you get simple but effective in-cluster secret delivery. For more advanced use cases, **HashiCorp Vault** provides dynamic, scalable, and secure secrets storage with full policy control.

By following these practices, you can build **secure, production-ready Docker applications** that avoid common pitfalls like leaked credentials or misconfigured access control.
