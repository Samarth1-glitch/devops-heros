# Kubernetes Services — Service Discovery & External Access

## Overview

This session covered the major Kubernetes Service types and how they provide

internal communication, external access, DNS-based discovery, and direct Pod discovery.

The practicals covered:

1\. ClusterIP

2\. NodePort

3\. LoadBalancer

4\. ExternalName

5\. Headless Service

# ClusterIP Service — Internal Microservice Communication

![alt text](<Screenshots/Screenshot 2026-09-17 115358.png>)

![alt text](<Screenshots/Screenshot 2026-09-17 115705.png>)


# NodePort Service — External Host-Level Cluster Access

![alt text](<Screenshots/Screenshot 2026-09-17 115751.png>)

# LoadBalancer Service — Production Public Cloud Ingress

![alt text](<Screenshots/Screenshot (92).png>)

# Headless Service (`clusterIP: None`) — Direct Pod-to-Pod Discovery

![alt text](<Screenshots/Screenshot 2026-09-17 120807.png>)
---

![alt text](<Screenshots/Screenshot 2026-09-17 120829.png>)

# All Services

![alt text](<Screenshots/Screenshot 2026-09-17 121212.png>)



# Conclusion

This session demonstrated how Kubernetes Services provide different networking

and service-discovery mechanisms.

The main distinction is:

    ClusterIP      → Internal Service Access

    NodePort       → Node-Level External Access

    LoadBalancer   → External Load Balancer

    ExternalName   → External DNS Alias

    Headless       → Direct Pod Discovery

Together, these Service types form the foundation for communication and

exposure of workloads in Kubernetes.

# Additional Questions & Research

## Q1. Difference Between DaemonSet and StatefulSet

| Feature         | DaemonSet                                | StatefulSet                                    |
| --------------- | ---------------------------------------- | ---------------------------------------------- |
| Primary purpose | Runs a Pod on every or selected node     | Runs Pods with stable identities               |
| Pod placement   | Node-oriented                            | Scheduler-oriented                             |
| Pod identity    | Not stable                               | Stable and predictable                         |
| Pod names       | Random/generated                         | Ordered, e.g. `app-0`, `app-1`                 |
| Storage         | No special storage identity              | Can provide stable PersistentVolumeClaims      |
| Scaling         | Usually tied to number of matching nodes | Explicit replica count                         |
| Typical use     | Monitoring, logging, node agents         | Databases, Kafka, distributed stateful systems |

A **DaemonSet** is used when a workload needs a copy of a Pod on each matching node, such as log collectors or node-monitoring agents. A **StatefulSet** is used when Pods need stable identities, persistent storage, or ordered deployment/scaling.

### Simple way to remember

```text
DaemonSet  →  "I need one Pod per Node."

StatefulSet →  "I need Pods with identities."
```

---

## Q2. Difference Between Deployment, ReplicaSet and DaemonSet

| Feature         | Deployment                    | ReplicaSet                        | DaemonSet                       |
| --------------- | ----------------------------- | --------------------------------- | ------------------------------- |
| Main purpose    | Manage stateless applications | Maintain a fixed number of Pods   | Run a Pod on each matching node |
| Replica control | Yes                           | Yes                               | Based on nodes                  |
| Rolling updates | Yes                           | No built-in rollout orchestration | Yes                             |
| Pod identity    | Interchangeable               | Interchangeable                   | Node-associated                 |
| Typical use     | Web/API applications          | Usually managed by Deployment     | Node-level agents               |
| Example         | Frontend with 5 replicas      | Maintain 5 identical Pods         | Log collector on every node     |

### Relationship

A Deployment normally manages a ReplicaSet, and the ReplicaSet manages the actual Pods:

```text
Deployment
    |
    ↓
ReplicaSet
    |
    +---- Pod
    +---- Pod
    +---- Pod
```

A DaemonSet works differently:

```text
DaemonSet
    |
    +---- Node 1 → Pod
    +---- Node 2 → Pod
    +---- Node 3 → Pod
```

A **Deployment** is generally the appropriate abstraction for stateless applications and provides declarative updates and rolling deployments. A **ReplicaSet** primarily maintains the desired number of identical Pods and is normally managed by a Deployment. A **DaemonSet** ensures a Pod runs on every or selected node.

---

## Q3. Research: CoreDNS and Kubernetes DNS

**CoreDNS** is the default DNS implementation used for Kubernetes cluster DNS.

It allows Pods to discover Services and other Kubernetes resources using DNS names instead of hardcoded IP addresses.

For example:

```text
web-service-clusterip
        ↓
CoreDNS
        ↓
Service ClusterIP
```

For a Headless Service:

```text
web-service-headless
        ↓
CoreDNS
        ↓
Pod IPs
```

For an `ExternalName` Service:

```text
external-database-service
        ↓
CoreDNS
        ↓
CNAME → api.github.com
```

### Why CoreDNS matters

* Provides DNS-based Service Discovery.
* Allows applications to use stable Service names instead of IP addresses.
* Resolves Kubernetes Service names such as:

```text
<service>.<namespace>.svc.cluster.local
```

* Supports Headless Service discovery by returning individual Pod addresses.
* Can be configured through its CoreDNS `Corefile`/ConfigMap.

### NodeLocal DNSCache

Kubernetes also supports **NodeLocal DNSCache**, which runs a DNS caching agent as a DaemonSet on cluster nodes.

The goal is to improve DNS performance and reduce reliance on cross-node DNS traffic, DNAT, and connection tracking.

```text
Pod
 ↓
NodeLocal DNSCache
 ↓
CoreDNS
 ↓
Kubernetes / External DNS
```

---

## Quick Revision

```text
Deployment
→ Manages stateless application releases.

ReplicaSet
→ Maintains the required number of identical Pods.

DaemonSet
→ Runs a Pod on every or selected Node.

StatefulSet
→ Runs Pods with stable identities and storage relationships.

CoreDNS
→ Provides DNS-based service discovery inside Kubernetes.
```