---
layout: post
title: Kubernetes Secrets Management Using Vault and Kubernetes for Secure Access
subtitle: Explore advanced techniques to manage secrets securely in Kubernetes clusters leveraging HashiCorp Vault and Kubernetes native features
categories: Kubernetes
tags: [Kubernetes, Secrets Management, Vault, HashiCorp Vault, Security, DevOps, Cloud Native, Kubernetes Security]
excerpt: Discover how to securely manage secrets in Kubernetes using Vault and Kubernetes native secrets. This guide dives into advanced best practices and integrations for robust secret management in cloud native environments.
---
Managing secrets like API keys, passwords, and certificates securely is a critical challenge in Kubernetes environments. While Kubernetes provides a native **Secrets** resource, it has limitations in terms of encryption, access control, and audit capabilities. To address these gaps, integrating **HashiCorp Vault** with Kubernetes offers a robust solution for secrets management. This article explores how to leverage Vault alongside Kubernetes native mechanisms for **secure and scalable secrets management** in production-grade clusters.

#### Why Native Kubernetes Secrets May Not Be Enough

Kubernetes Secrets are stored in etcd and by default are **base64-encoded but not encrypted** unless you enable encryption at rest. Additionally, Kubernetes Secrets:

- Lack fine-grained access control beyond namespace scoping.
- Provide limited auditing for secret access.
- Are static and require manual rotation.

These limitations make Kubernetes Secrets insufficient for high-security environments requiring dynamic secrets and compliance controls.

#### Introducing HashiCorp Vault

**Vault** is a secrets management tool designed for dynamic secrets, encryption as a service, and tight access controls. It supports features like:

- Dynamic secret generation (e.g., database credentials).
- Fine-grained policies controlling secret access.
- Audit logging for all secret interactions.
- Automatic secrets rotation and leasing.

When integrated with Kubernetes, Vault can dynamically inject secrets into Pods, minimizing static secret exposure.

#### Kubernetes and Vault Integration Overview

##### Authentication with Kubernetes Service Accounts

Vault supports Kubernetes authentication by trusting the Kubernetes API server tokens attached to Pods. The integration flow involves:

1. Vault is configured to trust the Kubernetes cluster CA and the service account JWT.
2. A Kubernetes Pod requests a Vault token using its service account token.
3. Vault validates the token and returns a Vault token scoped with policies defining secret access.

##### Injecting Secrets into Pods

Two popular methods for delivering Vault secrets to Kubernetes workloads are:

- **Vault Agent Injector:** Sidecar container automatically injects secrets as files or environment variables.
- **Init Containers:** An init container fetches secrets from Vault before the main container starts.

This ensures secrets are dynamically retrieved at runtime, eliminating static secrets baked into container images or manifests.

#### Step-by-Step Example: Configuring Vault with Kubernetes

1. **Enable Kubernetes Auth in Vault**

```bash
vault auth enable kubernetes
```

2. **Configure the Kubernetes Auth method**

```bash
vault write auth/kubernetes/config \
    token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

3. **Create a Vault policy defining access**

```hcl
path "secret/data/myapp/*" {
  capabilities = ["read"]
}
```

4. **Bind the policy to a Kubernetes service account**

```bash
vault write auth/kubernetes/role/myapp-role \
    bound_service_account_names=myapp-sa \
    bound_service_account_namespaces=default \
    policies=myapp-policy \
    ttl=24h
```

5. **Deploy your application Pod with Vault annotations**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "myapp-role"
    vault.hashicorp.com/agent-inject-secret-config: "secret/data/myapp/config"
spec:
  serviceAccountName: myapp-sa
  containers:
  - name: myapp-container
    image: myapp:latest
```

This setup dynamically fetches secrets and injects them into the Pod without hardcoding sensitive data.

#### Kubernetes Native Secrets Encryption

While Vault provides a powerful external secrets manager, it is recommended to **enable encryption at rest for Kubernetes Secrets** as an additional layer:

- Configure `EncryptionConfiguration` on the API server.
- Use AES-CBC or AES-GCM providers.
- Rotate encryption keys regularly.

This protects native Secrets stored in etcd from unauthorized access.

#### Best Practices for Secure Secrets Management in Kubernetes

- **Prefer dynamic secrets** from Vault over static secrets.
- Use **short-lived tokens and leases** to minimize exposure.
- Implement **RBAC policies** tightly scoped to least privilege.
- Enable **audit logging** on Vault and Kubernetes API server.
- Regularly **rotate secrets and encryption keys**.
- Use **network policies** to restrict access to Vault and Kubernetes API.

#### Common Challenges and Troubleshooting

- **Token validation failures:** Ensure Vault trusts Kubernetes CA and service account tokens.
- **Secrets not injected:** Check Vault Agent Injector logs and Pod annotations.
- **Access denied:** Verify Vault policies and Kubernetes RBAC bindings.
- **Latency issues:** Optimize Vault performance or cache secrets if needed.

#### Conclusion

Combining **HashiCorp Vault** with Kubernetes native Secrets provides a robust, scalable, and secure secrets management strategy. This hybrid approach balances the ease of Kubernetes Secrets with Vault’s dynamic capabilities and fine-grained control. Mastering this integration empowers DevOps and security teams to safeguard sensitive data effectively across cloud-native environments.

