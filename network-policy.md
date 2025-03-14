# Network Policy

In Kubernetes, a **NetworkPolicy** is a set of rules that control the traffic flow between **pods** within a cluster, and between pods and other network endpoints (such as external services or the internet). It is a resource in Kubernetes used to define how groups of pods are allowed to communicate with each other, helping to enforce security and traffic control at the network level.

NetworkPolicies are important for securing communication in a Kubernetes cluster by restricting which pods can access which others. Without a NetworkPolicy, all pods in a Kubernetes cluster can communicate freely with each other by default (assuming no other network restrictions or firewalls).

### Key Concepts of Kubernetes NetworkPolicy:

1. **Pod Selector:** 
   - A **pod selector** is used to define which pods the NetworkPolicy applies to. This selector can target pods based on labels, allowing fine-grained control over which pods are affected by the policy.

2. **Ingress and Egress Rules:**
   - **Ingress** rules define the allowed incoming traffic to a pod.
   - **Egress** rules define the allowed outgoing traffic from a pod.
   
   Both rules allow specifying allowed or denied traffic based on various parameters such as:
   - Source/Destination IP addresses
   - Source/Destination ports
   - Pod selectors
   - Namespace selectors
   - Protocol types (TCP, UDP)

3. **Default Behavior:**
   - By default, if no NetworkPolicy is defined, **all pods can communicate with each other** (the cluster allows all ingress and egress traffic).
   - Once a NetworkPolicy is created, the behavior changes to enforce only the allowed communication patterns defined in the policy. If no ingress or egress rules are specified, all traffic is denied by default.

4. **Policy Types:**
   A NetworkPolicy can include one or more of the following policy types:
   - `Ingress`: Controls incoming traffic to the pods.
   - `Egress`: Controls outgoing traffic from the pods.

### Example of a Simple NetworkPolicy

Here’s a basic example of a NetworkPolicy that allows incoming traffic to a pod only from other pods in the same namespace with a specific label.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

In this example:
- The `podSelector: {}` indicates that the policy applies to all pods in the namespace.
- The `ingress` rule allows incoming traffic only from pods labeled `app: frontend`.
- No egress rule is defined, so outgoing traffic would be allowed by default unless a policy is defined.

### Key Points to Remember:

1. **NetworkPolicy only works with a CNI plugin** that supports network policies (such as Calico, Cilium, or Weave).
2. Once a **NetworkPolicy** is applied, traffic that isn't explicitly allowed is **denied**.
3. NetworkPolicies are namespace-scoped, meaning they apply to pods within a specific namespace.

Network policies provide a powerful way to secure communication between services within your Kubernetes cluster, limiting unnecessary exposure and controlling access based on labels and other network attributes.

Solutions that support Network policies:

- kube-router
- Calico
- Romana
- Weave-net
  
Solutions that do not support network policies:
- Flannel


```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080

  - ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53

```

