---
marp: true
theme: default
class:
---

# Argo CD 1.8 Performance improvements

![bg right contain](https://raw.githubusercontent.com/argoproj/argoproj/master/docs/assets/argo.png)

---

# Controller Sharding

* Controller deployment converted to StatefulSet.
* Each controller instance is responsible for one shard. Shard is a subset of clusters and related Argo CD applications.
* Cluster Shard:
```
clusterShard = clusterSecret.UID % replicas count
```

![bg right contain](https://user-images.githubusercontent.com/426437/98052637-8e7cdd80-1deb-11eb-91e0-373b88a996b9.png)


---

# Controller Sharding (Controller Stateful Set)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: argocd-application-controller
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: argocd-application-controller
        env:
        - name: ARGOCD_CONTROLLER_REPLICAS
          value: "2"
```

---

# Concurrent Manifest Generation

![bg right contain](https://user-images.githubusercontent.com/426437/98054733-9f7c1d80-1df0-11eb-8d5a-3924b1690c16.png)


**Before:**

* One concurrent manifest generation request per repository per repo-server instance
* Repo initialization ( `git fetch`, `helm dep up` ) executed for each manifest generation request

---

# Concurrent Manifest Generation

**After:**

![bg right contain](https://user-images.githubusercontent.com/426437/98054771-bf134600-1df0-11eb-9037-e434b18fbc63.png)


* Manifets generation requests with the **same repository and same revision** are processed concurrently in batch
* One repo initialization ( `git fetch`, `helm dep up` )  for each batch

---

# Path Manifest Generation Annotation

## Use case

Mono-repository with multiple unrelated applications in the same branch.

## Annotation `argocd.argoproj.io/manifest-generate-paths`

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  annotations:
    argocd.argoproj.io/manifest-generate-paths: .
spec:
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    path: guestbook
```

---

# User Interface improvements

![image](https://user-images.githubusercontent.com/426437/98055626-cb000780-1df2-11eb-9d82-80d8dbee2475.png)
