# ☸️ Kubernetes — Pods, Services, ReplicaSets & Deployments

A beginner-friendly guide to understanding Kubernetes concepts with real commands and simple explanations.

---

## 📐 How Kubernetes Works — Simple Explanation

Think of Kubernetes like a **restaurant**:

- 🧑‍💼 **Master Node** = The Manager — gives orders, doesn't cook. No pods run here.
- 👨‍🍳 **Worker Node (Slave)** = The Kitchen — all the actual work (pods) happens here.
- 📦 **Pod** = A dish being prepared — your app runs inside a pod.
- 🌐 **Service** = The waiter — connects the customer (user) to the right dish (pod).

```
┌─────────────────────────────────────┐
│           Master Node               │
│     (Boss — gives instructions)     │
│  kubeadm | kubectl | API Server     │
└──────────────┬──────────────────────┘
               │
   ┌───────────▼───────────┐
   │     Worker Node       │
   │  (Where pods live)    │
   │  kubelet | kube-proxy │
   └───────────────────────┘
```

### Networking — How Pods Talk to Each Other

| Component | What it does (Simply) |
|-----------|----------------------|
| **Cluster IP** | Like an internal phone number — pods call each other using this |
| **NodePort** | Like a door to the outside — lets users access your app from the internet |
| **Kube-Proxy** | The receptionist — gives each service an IP and port number |
| **CNI / Calico** | The roads between pods — handles all the networking between them |

---

## ⚡ Basic kubectl Commands (with Simple Explanation)

### 🔍 Check Nodes

```bash
kubectl get nodes
```
> **"Show me all the machines in my cluster."**
> You'll see the master and worker nodes with status `Ready` (healthy) or `NotReady` (something is wrong).

```bash
kubectl get nodes -o wide
```
> **"Show me more details about all machines."**
> Same as above, but also shows IP addresses and what OS/runtime each node is using.

---

### 🐳 Check Pods

```bash
kubectl get pods
```
> **"Show me all running apps (pods)."**
> You'll see pod names and their status — `Running` means it's working, `Pending` means it's waiting, `CrashLoopBackOff` means something is broken.

```bash
kubectl get pods -o wide
```
> **"Show me pods with more details."**
> Also tells you which worker node the pod is running on and what IP it has. Very helpful when you have multiple nodes.

```bash
kubectl describe pod <pod-name>
```
> **"Give me the full story of this pod."**
> Shows everything — what image it's using, what happened recently, and any error messages. When a pod is not working, look at the **Events** section at the bottom — it tells you exactly what went wrong.

```bash
kubectl delete pod <pod-name>
```
> **"Kill this pod."**
> If the pod is part of a Deployment or ReplicaSet, Kubernetes will automatically create a new one to replace it. It's like Kubernetes saying "don't worry, I'll make another one."

---

### 🚀 Create & Manage Deployments

```bash
kubectl create deployment nginx --image=nginx --replicas=2
```
> **"Start an nginx app and keep 2 copies of it running."**
> Kubernetes will launch 2 pods running nginx. If one dies, it brings it back automatically.

```bash
kubectl get deployment
```
> **"Show me all my deployments."**
> You'll see how many pods are desired vs how many are actually running and ready.

```bash
kubectl get all
```
> **"Show me everything in my cluster."**
> Displays all pods, deployments, replicasets, and services in one shot. Great for a full overview.

```bash
kubectl delete all --all
```
> ⚠️ **"Delete EVERYTHING."**
> Removes all pods, services, deployments — everything in the current namespace. Use this carefully — there's no undo!

---

### 🌐 Expose Your App (Create a Service)

```bash
kubectl expose deployment nginx --type=NodePort --port=80
```
> **"Create a door so people from the internet can reach my nginx app."**
> This creates a Service with a random port (between 30000–32767). You then open that port in your EC2 Security Group and visit `http://<worker-ip>:<that-port>` in a browser.

```bash
kubectl get svc
```
> **"Show me all services."**
> Look at the `PORT(S)` column. You'll see something like `80:31500/TCP`. The `31500` is the port you access from outside. The `80` is what the container uses internally.

---

### 📈 Scale Your App

```bash
kubectl scale deployment nginx --replicas=5
```
> **"I want 5 copies of my app running, not 2."**
> Kubernetes will create 3 more pods (since 2 already exist). Scaling down works the same way — just set a lower number and extra pods are removed cleanly.

---

### 🔌 Port Types — Simple Explanation

| Term | Simple Meaning |
|------|----------------|
| `containerPort` | The port your app listens on **inside** the container (e.g., nginx listens on 80) |
| `targetPort` | Same as containerPort — where the Service forwards traffic **to** |
| `port` | The port the Service itself listens on **inside** the cluster |
| `NodePort` | The port you use to access your app **from the internet** (30000–32767) |

---

## 📄 YAML Files — What They Mean

### 1. Pod — `pod.yml`

> A Pod is the smallest unit in Kubernetes. Think of it as a single running copy of your app.

```yaml
apiVersion: v1          # Which version of Kubernetes API to use
kind: Pod               # We are creating a Pod
metadata:
  name: nginx-pod       # Name of this pod
  labels:
    env: dev            # A tag/sticker on this pod — Service uses this to find it
spec:
  containers:
    - name: jd          # Name of the container inside the pod
      image: nginx      # Docker image to use (downloads from Docker Hub)
      ports:
        - containerPort: 80   # The port our app listens on inside the container
```

```bash
kubectl apply -f pod.yml
```
> **"Create this pod from the file."**
> Kubernetes reads the file, understands what you want, and creates the pod on a worker node.

```bash
kubectl get pods -o wide
```
> **"Is my pod running? Which node is it on?"**

```bash
kubectl describe pod nginx-pod
```
> **"Tell me everything about this pod — especially if something is wrong."**
> Scroll to the bottom and read the Events section for error messages.

---

### 2. Service — `service.yml`

> A Service is like a **permanent address** for your pods. Pods come and go, but the Service always stays and routes traffic to the right place.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service       # Name of this service
spec:
  selector:
    env: dev                # Find all pods that have the label "env: dev" and send traffic to them
  ports:
    - protocol: TCP
      port: 80              # Port the service listens on (inside the cluster)
      targetPort: 80        # Port on the container to forward traffic to
  type: NodePort            # Make this app accessible from outside the cluster
```

```bash
kubectl apply -f service.yml
```
> **"Create this service."**
> The service finds pods with the label `env: dev` and starts routing traffic to them.

```bash
kubectl get svc
```
> **"Show me all services and their ports."**
> Note the NodePort assigned (e.g., `31500`). Open that port in your EC2 Security Group, then visit `http://<worker-public-ip>:31500` in your browser.

> ⚠️ **Don't forget:** Add the NodePort to your EC2 Security Group inbound rules, otherwise the browser won't connect.

---

### 3. ReplicationController — `rc.yml`

> Think of this as a **security guard** — its only job is to make sure a fixed number of pods are always running. If one pod dies, it creates a new one immediately.
> This is the **older way** of doing it. ReplicaSet replaced it.

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
spec:
  replicas: 3           # Always keep 3 pods running
  selector:
    app: nginx          # Manage pods that have the label "app: nginx"
  template:             # Blueprint for creating new pods
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f rc.yml
```
> **"Start the ReplicationController and create 3 pods."**

```bash
kubectl get pods
```
> **"Are my 3 pods running?"**
> Try deleting one with `kubectl delete pod <name>` — you'll see Kubernetes creates a new one automatically.

```bash
kubectl get rc
```
> **"Show me my ReplicationController status."**

```bash
kubectl delete rc nginx-rc
```
> **"Delete the RC and all its pods."**
> Important: If you only delete individual pods, the RC will keep recreating them. You must delete the RC itself to stop everything.

---

### 4. ReplicaSet — `rs.yml`

> Same idea as ReplicationController — keeps a fixed number of pods running — but **smarter**. It supports better label matching and is the modern standard.

#### Option A — Simple Label Match (`matchLabels`)

> "Manage pods that have exactly this label."

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx          # Only manage pods with label "app: nginx"
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

#### Option B — Advanced Label Match (`matchExpressions`)

> "Manage pods where the label value is one of these options." More flexible than matchLabels.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - nginx         # Manage pods labeled "app: nginx"
          - mysql         # OR "app: mysql"
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f rs.yml
```
> **"Create the ReplicaSet and start 3 pods."**

```bash
kubectl get pods
```
> **"Confirm 3 pods are running."**

```bash
kubectl get rs
```
> **"Show ReplicaSet health — DESIRED vs CURRENT vs READY."**
> All three numbers should match when everything is working fine.

---

### 5. Deployment — `deployment.yml`

> A Deployment is the **best and recommended way** to run apps. It does everything a ReplicaSet does, plus it lets you **update your app without downtime** and **roll back if something breaks**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydeploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: jd-pod         # Manage pods with this label
  template:
    metadata:
      labels:
        app: jd-pod
    spec:
      containers:
        - name: my-container
          image: nginx      # Start with nginx
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f deployment.yml
```
> **"Create the Deployment."**
> Behind the scenes, Kubernetes creates a ReplicaSet, which then creates 3 pods. You manage the Deployment and Kubernetes handles the rest.

```bash
kubectl get deployment
```
> **"Is my Deployment healthy?"**
> Check READY column — it should show `3/3` when all pods are up.

```bash
kubectl get all
```
> **"Show everything — Deployment, ReplicaSet, and Pods together."**
> Great way to see how all three are connected.

---

## 🔄 Rolling Updates & Rollbacks

> This is where Deployments shine. You can update your app live without shutting anything down.

### Update the App Image

```bash
kubectl set image deployment mydeploy my-container=httpd
```
> **"Switch from nginx to httpd (Apache) without downtime."**
> Kubernetes replaces pods one by one — old pod goes down, new pod comes up, repeat. Your app stays accessible throughout.

```bash
kubectl set image deployment mydeploy my-container=nginx && kubectl rollout status deployment mydeploy --watch
```
> **"Update to nginx and show me the live progress."**
> You'll watch each pod being replaced in real time in your terminal. When you see `successfully rolled out`, the update is done. Press `Ctrl+C` to stop watching.

---

### Check History

```bash
kubectl rollout history deployment mydeploy
```
> **"Show me all previous versions of my deployment."**
> Every time you update the image, a new revision is saved. You'll see a numbered list like Revision 1, 2, 3... This is your safety net.

```bash
kubectl describe pod <pod-name>
```
> **"What image is this pod actually running right now?"**
> Look for the `Image:` line in the output to confirm.

---

### Undo / Rollback

```bash
kubectl rollout undo deployment mydeploy
```
> **"Something broke — take me back to the previous version!"**
> If you updated from nginx → httpd and it's not working, this one command switches everything back to nginx. Safe and instant.

```bash
kubectl rollout history deployment mydeploy
```
> **"Confirm the rollback worked."**
> Run this after the undo — you'll see the revision numbers updated, confirming you're back on the previous version.

---

## 🗂️ File Summary

| File | Type | What it does |
|------|------|--------------|
| `pod.yml` | Pod | Runs a single nginx container |
| `service.yml` | Service | Opens a door so users can access your app |
| `rc.yml` | ReplicationController | Old way to keep pods running (3 replicas) |
| `rs.yml` | ReplicaSet | Modern way to keep pods running, smarter labels |
| `deployment.yml` | Deployment | Best way — run, update, and rollback your app |

---

## 🔑 RC vs ReplicaSet vs Deployment — Which to Use?

| Feature | ReplicationController | ReplicaSet | Deployment |
|---------|:--------------------:|:----------:|:----------:|
| Keeps pods running | ✅ | ✅ | ✅ |
| Smart label matching | ❌ | ✅ | ✅ |
| Update app without downtime | ❌ | ❌ | ✅ |
| Rollback to old version | ❌ | ❌ | ✅ |
| Should you use it? | ❌ Old, avoid | ⚠️ Rarely alone | ✅ Always |

> 💡 **Simple rule:** Always use **Deployment** in real projects. It does everything the others do, plus more.

---

## 🧰 Tech Stack

- **Kubernetes:** v1.29
- **Container Runtime:** containerd
- **CNI:** Calico
- **Images Used:** `nginx`, `httpd`

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).
