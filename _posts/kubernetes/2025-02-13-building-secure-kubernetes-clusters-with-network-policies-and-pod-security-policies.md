---
layout: post
title: Building Secure Kubernetes Clusters with Network Policies and Pod Security Policies
subtitle: Strengthen your Kubernetes cluster security using advanced Network Policies and Pod Security Policies
categories: Kubernetes
tags: [Kubernetes, NetworkPolicy, PodSecurityPolicies, ClusterSecurity, DevOps, Cloud Security, Container Security]
excerpt: Explore how to secure Kubernetes clusters by implementing Network Policies for traffic control and Pod Security Policies for enforcing pod-level security best practices.
---
Securing Kubernetes clusters is critical to maintaining robust, reliable, and compliant containerized environments. Two powerful features — **Network Policies** and **Pod Security Policies (PSPs)** — provide essential layers of defense to control network traffic and enforce strict pod security standards. This post targets intermediate to advanced Kubernetes users looking to deepen their security expertise and build hardened clusters.

---

#### Understanding Kubernetes Network Policies

By default, Kubernetes allows unrestricted network communication between pods within a cluster. This default openness poses significant risks in multi-tenant or sensitive environments.

##### What Are Network Policies?

Network Policies are Kubernetes resources that act as *firewalls* at the pod level, controlling **ingress** and **egress** traffic based on labels and namespaces. They are enforced by the cluster’s network plugin (CNI), such as Calico, Cilium, or Weave.

##### Core Components of Network Policies

- **Pod Selector:** Specifies which pods the policy applies to.
- **Ingress and Egress Rules:** Define allowed sources and destinations.
- **Policy Types:** You can define ingress-only, egress-only, or both.

##### Example: Default Deny All Ingress Policy

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

This policy blocks *all* inbound traffic to pods in the `production` namespace unless explicitly allowed by other policies.

##### Best Practices

- Always start with a **default deny all** baseline for ingress and egress.
- Use **granular labels** for pods to apply precise policies.
- Combine **namespace selectors** with pod selectors to limit cross-namespace communication.
- Continuously test policies to avoid unintended service disruptions.

---

#### Enforcing Pod Security with Pod Security Policies (PSPs)

*Note: Kubernetes is deprecating PSPs in favor of Pod Security Admission Controllers, but many clusters still rely on PSPs.*

##### What Are Pod Security Policies?

PSPs are cluster-level resources that define a set of conditions that a pod must meet to be admitted. They control pod attributes such as:

- Privileged mode
- Host networking and ports
- Use of volume types
- Linux capabilities
- Running as root user or specific user IDs

##### Example: Restrictive Pod Security Policy

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  volumes:
  - configMap
  - emptyDir
  - secret
  - persistentVolumeClaim
  allowedCapabilities: []
  runAsUser:
    rule: MustRunAsNonRoot
  seLinux:
    rule: RunAsAny
  fsGroup:
    rule: MustRunAs
    ranges:
    - min: 1
      max: 65535
  readOnlyRootFilesystem: true
```

##### Binding PSPs with RBAC

To enforce PSPs, you must create `Role` or `ClusterRole` and bind them to users or service accounts:

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: psp-restricted
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs: ['use']
  resourceNames:
  - restricted
```

---

#### Transition to Pod Security Admission

With PSP deprecation planned, Kubernetes recommends adopting the **Pod Security Admission Controller**, which uses predefined profiles (privileged, baseline, restricted) applied per namespace.

##### Advantages of Pod Security Admission

- Easier to configure via namespace labels.
- Less complex than PSP RBAC bindings.
- Native support and ongoing development.

---

#### Integrating Network Policies and Pod Security for Holistic Cluster Security

- **Network Policies** prevent unauthorized lateral movement between pods.
- **Pod Security Policies** enforce secure pod configurations to limit attack vectors.
- Together, they form a **defense-in-depth** approach, essential for compliance and operational security.

---

#### Monitoring and Auditing

- Use tools like **Calico Enterprise**, **Cilium Hubble**, or **Weave Scope** for real-time network visibility.
- Leverage Kubernetes **audit logs** to detect policy violations.
- Regularly review and update policies to adapt to evolving workloads and threat landscapes.

---

#### Conclusion

Building secure Kubernetes clusters requires a combination of **network segmentation** and **pod-level security controls**. Mastering Network Policies and Pod Security Policies empowers cluster administrators to reduce attack surfaces and enforce consistent security postures. As the Kubernetes ecosystem evolves, transitioning to newer security admission mechanisms will ensure your cluster stays resilient and compliant.

