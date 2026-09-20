# Kubernetes Deployment Strategies

This project demonstrates three important Kubernetes deployment strategies:

1. Blue-Green Deployment
2. Canary Deployment
3. Recreate Deployment

Each strategy is implemented using Kubernetes Deployments and Services and tested on Minikube.

---

# Strategy Comparison

| Strategy | Downtime | Resource Cost | Rollback Speed | Best Use Case |
|-----------|-----------|-----------|-----------|-----------|
| Rolling Update | No | Low | Fast | Stateless applications |
| Blue-Green | No | High (2x resources) | Instant | Critical production releases |
| Canary | No | Low | Fast | Gradual production validation |
| Recreate | Yes | Lowest | Moderate | Database migrations, RWO volumes |

---

# 1. Blue-Green Deployment

## What is Blue-Green Deployment?

Blue-Green Deployment maintains two identical production environments:

- **Blue** = Current live version
- **Green** = New version being prepared

Traffic is switched instantly by changing the Kubernetes Service selector.

---

## Architecture

```text
Users
   |
   v
[Service]
   |
   +----> BLUE (v1)   <-- LIVE
   |
   +----> GREEN (v2)  <-- STANDBY
```

After switch:

```text
Users
   |
   v
[Service]
   |
   +----> GREEN (v2)  <-- LIVE
   |
   +----> BLUE (v1)   <-- STANDBY
```

---

## Deploy Blue and Green Environments

### Command

```bash
kubectl apply -f 02-blue-green/deployment-blue.yaml
kubectl apply -f 02-blue-green/deployment-green.yaml

kubectl get pods -l app=myapp --show-labels
```

---

## Make BLUE Live

### Command

```bash
kubectl apply -f 02-blue-green/service-blue.yaml

minikube service myapp-service --url
```

### Result

BLUE environment becomes active.

### Screenshot

![alt text](Screenshots/blue-green(1).png)

---

## Switch Traffic to GREEN

### Command

```bash
kubectl apply -f 02-blue-green/service-green.yaml

minikube service myapp-service --url
```

### Result

Traffic is instantly redirected to Green.

### Screenshot

![alt text](Screenshots/blue-green(2).png)

---

## Rollback to BLUE

### Command

```bash
kubectl apply -f 02-blue-green/service-blue.yaml
```

### Result

Traffic instantly returns to Blue.

![alt text](Screenshots/blue-green(3).png)

---

## Cleanup

```bash
kubectl delete -f 02-blue-green/service-blue.yaml
kubectl delete -f 02-blue-green/deployment-blue.yaml
kubectl delete -f 02-blue-green/deployment-green.yaml
```

---

# 2. Canary Deployment

## What is Canary Deployment?

Canary Deployment releases a new version to a small percentage of users before full rollout.

Example:

```text
9 Stable Pods (v1)
1 Canary Pod (v2)

≈ 90% traffic → Stable
≈ 10% traffic → Canary
```

---

## Architecture

```text
                    [SERVICE]
                         |
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
    Stable v1       Stable v1       Canary v2

Traffic Distribution:
90% -> Stable (v1)
10% -> Canary (v2)
```

---

## Deploy Stable Version

### Command

```bash
kubectl apply -f 03-canary/deployment-stable.yaml

kubectl rollout status deployment/app-stable

kubectl apply -f 03-canary/service.yaml
```

---

## Deploy Canary Version

### Command

```bash
kubectl apply -f 03-canary/deployment-canary.yaml

kubectl get pods -l app=myapp-canary --show-labels
```

### Screenshot

![alt text](Screenshots/canary(1).png)

---

## Increase Canary Traffic

### Command

```bash
kubectl scale deployment app-canary --replicas=3

kubectl scale deployment app-stable --replicas=7

kubectl get endpoints myapp-canary-service
```

### Result

Traffic distribution becomes approximately:

- 70% Stable
- 30% Canary

### Screenshot

![alt text](Screenshots/canary(2).png)

---

## Promote Canary to Production

### Command

```bash
kubectl scale deployment app-canary --replicas=9

kubectl scale deployment app-stable --replicas=0
```

### Remove Old Stable Deployment

```bash
kubectl delete deployment app-stable
```

---

## Rollback Canary

### Command

```bash
kubectl scale deployment app-canary --replicas=0

kubectl scale deployment app-stable --replicas=9
```

---

## Cleanup

```bash
kubectl delete -f 03-canary/service.yaml

kubectl delete -f 03-canary/deployment-canary.yaml

kubectl delete -f 03-canary/deployment-stable.yaml
```

---

# 3. Recreate Deployment

## What is Recreate Deployment?

Recreate Deployment completely shuts down the old version before starting the new version.

```text
[v1] [v1] [v1]
      |
      v
All Pods Terminated
      |
      v
*** DOWNTIME ***
      |
      v
[v2] [v2] [v2]
```

---

## Deploy Version 1

### Command

```bash
kubectl apply -f 04-recreate/deployment-v1.yaml

kubectl apply -f 04-recreate/service.yaml

kubectl get pods -l app=app-recreate
```

---

## Verify Service

### Command

```bash
minikube service app-recreate-service --url
```

### Screenshot

![alt text](Screenshots/recreate(1).png)

---

## Trigger Recreate Update

### Terminal 1

```bash
kubectl get pods -l app=app-recreate -w
```

### Terminal 2

```bash
kubectl apply -f 04-recreate/deployment-v2.yaml
```

### Screenshot

![alt text](Screenshots/recreate(2).png)

### Screenshot

![alt text](Screenshots/recreate(3).png)

### Observation

- Existing v1 pods terminate first.
- No pods are available temporarily.
- New v2 pods are created afterward.
- A short downtime window occurs.

---

## Verify Version 2

### Command

```bash
minikube service app-recreate-service --url
```

### Screenshot

![alt text](Screenshots/recreate(4).png)

---

## Rollback

### Command

```bash
kubectl rollout undo deployment/app-recreate

kubectl rollout status deployment/app-recreate
```

### Output

```text
deployment.apps/app-recreate rolled back

deployment "app-recreate" successfully rolled out
```
![alt text](Screenshots/recreate(5).png)

---

## Cleanup

```bash
kubectl delete -f 04-recreate/service.yaml

kubectl delete -f 04-recreate/deployment-v2.yaml
```

---

# Key Learnings

## Blue-Green Deployment

### Advantages

- Zero downtime deployment
- Instant rollback
- Complete production testing before cutover

### Disadvantages

- Requires double infrastructure
- Higher resource consumption

---

## Canary Deployment

### Advantages

- Gradual rollout
- Real user validation
- Reduced production risk

### Disadvantages

- Traffic distribution is approximate
- Requires monitoring and metrics

---

## Recreate Deployment

### Advantages

- Simple deployment process
- Useful for schema migrations
- Works with ReadWriteOnce volumes

### Disadvantages

- Causes downtime
- Not suitable for highly available applications

---

# Conclusion

This project demonstrates three widely used Kubernetes deployment strategies:

- **Blue-Green Deployment** for instant switching and rollback.
- **Canary Deployment** for gradual and controlled releases.
- **Recreate Deployment** for applications that cannot run multiple versions simultaneously.

Understanding these strategies helps DevOps engineers choose the appropriate deployment model based on availability requirements, rollback needs, infrastructure costs, and application architecture.

---

## Author

**Samarth Patil**