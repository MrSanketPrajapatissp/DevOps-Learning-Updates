# Kubernetes — Complete DevOps Interview & Practical Guide

> **How to use this guide:** Topics flow from beginner → advanced. Every section has (1) what to say in an interview, (2) plain-English explanation with a real-world example, (3) the original faculty notes, and (4) ready-to-execute commands and YAML manifests in copy-paste blocks.

---

## Table of Contents

1. [Docker Swarm vs Kubernetes](#1-docker-swarm-vs-kubernetes)
2. [Namespaces](#2-namespaces)
3. [Pods, Deployments & Services — Basic Workflow](#3-pods-deployments--services--basic-workflow)
4. [Services — ClusterIP, NodePort, LoadBalancer](#4-services--clusterip-nodeport-loadbalancer)
5. [Pod Failure States — ImagePullBackOff & CrashLoopBackOff](#5-pod-failure-states--imagepullbackoff--crashloopbackoff)
6. [Environment Variables (ENV & ENVFROM)](#6-environment-variables-env--envfrom)
7. [ConfigMaps](#7-configmaps)
8. [Secrets](#8-secrets)
9. [Resource Quota & Resource Limits](#9-resource-quota--resource-limits)
10. [Metrics Server](#10-metrics-server)
11. [Horizontal Pod Autoscaler (HPA)](#11-horizontal-pod-autoscaler-hpa)
12. [Persistent Volume (PV) & Persistent Volume Claim (PVC)](#12-persistent-volume-pv--persistent-volume-claim-pvc)
13. [Multi-Container Pods](#13-multi-container-pods)
14. [Sidecar Design Pattern (with Adapter, Ambassador, Init Container)](#14-sidecar-design-pattern-with-adapter-ambassador-init-container)
15. [Ingress](#15-ingress)
16. [Pod Scheduling — Node Selector, Node Affinity, Taints & Tolerations](#16-pod-scheduling--node-selector-node-affinity-taints--tolerations)

---

## 1. Docker Swarm vs Kubernetes

### What Interviewers Expect

A strong DevOps engineer would say: _"Both are container orchestration platforms. Docker Swarm is Docker's native, lightweight option — easy to set up, good for small clusters. Kubernetes is the industry standard for production at scale, offering advanced scheduling, RBAC, network policies, and a massive ecosystem. For anything beyond a handful of services, Kubernetes is the right choice."_

### Beginner Explanation

**Orchestration** = automatically running, scaling, and connecting containers across many servers. **Real-world example:** A 5-service startup app may run fine on Swarm; a bank with 500+ microservices across multiple AWS regions needs Kubernetes for autoscaling, rolling updates, and security policies.

### Faculty Notes Explanation:

Docker Swarm and Kubernetes are both container orchestration platforms, but they have different strengths and use cases

Docker Swarm is easier to use and is better for smaller applications, while Kubernetes is more robust and better for large-scale applications.

Docker Swarm is Docker's native clustering and orchestration tool, designed to manage and scale containerized applications.

While it provides simplicity and integration with Docker, it has several limitations compared to other orchestration solutions like Kubernetes.

**Limitations**

Docker Swarm is generally considered less scalable than Kubernetes. While it can handle clusters with a reasonable number of nodes and services, Kubernetes is designed for larger-scale deployments and can manage thousands of nodes and pods.

Swarm provides basic load balancing and routing but lacks the advanced capabilities like provided by K8S

Kubernetes offers more granular security controls, including Role-Based Access Control (RBAC), network policies, and security policies. Docker Swarm's security features are less comprehensive in comparison.

Docker Swarm has a smaller community and less industry adoption compared to Kubernetes

---

## 2. Namespaces

### What Interviewers Expect

A strong DevOps engineer would say: _"Namespaces are virtual clusters inside a physical cluster. They isolate resources logically — typically used for separating dev, staging, and prod, or for multi-tenant teams. Each namespace can have its own RBAC, quotas, and network policies."_

### Beginner Explanation

Think of one big office building (cluster) divided into separate floors (namespaces). Each team gets a floor; what one team does doesn't affect another. **Real-world example:** A company creates `dev`, `qa`, and `prod` namespaces so a developer's broken pod in `dev` never impacts paying customers in `prod`.

### Faculty Notes Explanation:

Lets create pods in namespaces

### 🛠️ Practical Steps

**Step 1 — List existing namespaces:**

```bash
kubectl get ns
```

**Step 2 — Create a namespace called `dev`:**

```bash
kubectl create ns dev
```

**Step 3 — Verify it exists:**

```bash
kubectl get ns
```

Currently I am in default namespace, how to check

**Step 4 — Check current context (no namespace shown = default):**

```bash
kubectl config view
```

**Step 5 — Switch to the `dev` namespace:**

```bash
kubectl config set-context --current --namespace=dev
```

**Step 6 — Confirm the switch:**

```bash
kubectl config view
```

**Step 7 — Get pods (you'll only see pods in `dev`):**

```bash
kubectl get pods
```

**Step 8 — Create 3 test pods inside this namespace:**

```bash
kubectl run dev1 --image nginx
kubectl run dev2 --image nginx
kubectl run dev3 --image nginx
```

**Step 9 — Verify:**

```bash
kubectl get pods
```

---

## 3. Pods, Deployments & Services — Basic Workflow

### What Interviewers Expect

A strong DevOps engineer would say: _"A Pod is the smallest deployable unit, usually wrapping one container. A Deployment manages ReplicaSets to ensure the desired number of pod replicas are always running. A Service gives those pods a stable network endpoint since pod IPs change on restart."_

### Beginner Explanation

- **Pod** = a single running instance (like 1 app process).
- **Deployment** = a manager that keeps N copies of that pod alive.
- **Service** = a fixed phone number to reach those pods, even when they're restarted.

**Real-world example:** You deploy `nginx` with 3 replicas via a Deployment, then attach a Service so users always hit one URL regardless of which pod handles the request.

### Faculty Notes Explanation:

Basic commands to interact with deployments, services, pods and ingress.

### 🛠️ Practical — Common Commands

**Apply manifests:**

```bash
kubectl apply -f httpd.yml
kubectl apply -f nginx.yml
kubectl apply -f ingress.yml
```

**View all resources:**

```bash
kubectl get deploy
kubectl get svc
kubectl get pods
kubectl get ingress
```

**View ingress controller service (look for the EXTERNAL-IP / ELB):**

```bash
kubectl get services ingress-nginx-controller --namespace=ingress-nginx
```

---

## 4. Services — ClusterIP, NodePort, LoadBalancer

### What Interviewers Expect

A strong DevOps engineer would say: _"There are three main service types: ClusterIP exposes the service only inside the cluster (default), NodePort opens a port on every node (range 30000–32767) for external access, and LoadBalancer provisions an external cloud load balancer like an AWS ELB. Each maps incoming traffic to pods via label selectors."_

### Beginner Explanation

| Type             | Where it's reachable           | Use case                     |
| ---------------- | ------------------------------ | ---------------------------- |
| **ClusterIP**    | Inside cluster only            | Microservice-to-microservice |
| **NodePort**     | `<NodeIP>:<port>` from outside | Quick testing                |
| **LoadBalancer** | Cloud LB (e.g., AWS ELB URL)   | Production-facing app        |

**Real-world example:** Backend uses ClusterIP, frontend uses LoadBalancer (so users on the internet can reach it).

### 🛠️ Practical — LoadBalancer Service

**Step 1 — Create the manifest:**

```bash
vi service.yml
```

**Step 2 — Paste this:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mb-service
spec:
  type: LoadBalancer
  selector:
    app: bank
  ports:
    - port: 80
      targetPort: 80 # Must match container port. If omitted, K8s defaults to port 80.
      nodePort: 31433 # If omitted, K8s assigns a random port.
```

**Step 3 — Create the service:**

```bash
kubectl create -f service.yml
```

### 🛠️ Practical — NodePort Service

**Step 1 — Edit the manifest:**

```bash
vi service.yml
```

**Step 2 — Replace with:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mb-service
spec:
  type: NodePort # If port number is omitted, K8s assigns a random port.
  selector:
    app: bank
  ports:
    - port: 80
      targetPort: 80
```

**Step 3 — Apply (use `apply` to update existing resources):**

```bash
kubectl apply -f service.yml
```

**Step 4 — Inspect:**

```bash
kubectl describe svc mb-service
kubectl get svc
```

> **Important:** `http://IP:portnumber` won't work until you open the port in the AWS security group of `nodes.<cluster>.k8s.local`:
>
> - Custom TCP = `31433`, source = `anywhere`
> - All traffic, source = `anywhere`

### 🛠️ Practical — ClusterIP Service

**Step 1 — Create the manifest:**

```bash
vi service.yml
```

**Step 2 — Paste this:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ib-service
spec:
  type: ClusterIP
  selector:
    app: bank
  ports:
    - port: 80
```

---

## 5. Pod Failure States — ImagePullBackOff & CrashLoopBackOff

### What Interviewers Expect

A strong DevOps engineer would say: _"ImagePullBackOff happens when the kubelet can't pull the container image — usually a typo in the image name or missing imagePullSecret for a private registry. CrashLoopBackOff means the container starts but exits repeatedly — typically because of missing env variables, bad startup commands, or unmet dependencies. I troubleshoot using `kubectl describe pod` and `kubectl logs`."_

### Beginner Explanation

- **ImagePullBackOff** = "I can't even download the image."
- **CrashLoopBackOff** = "I downloaded it, but it keeps dying."

**Real-world example:** Deploying MariaDB without `MYSQL_ROOT_PASSWORD` → CrashLoopBackOff. Typing image as `nginxx` (typo) → ImagePullBackOff.

### Faculty Notes Explanation:

**Pod State: ImagePullBackOff:**

When a kubelet starts creating containers for a Pod using a container runtime, it might be possible the container is in Waiting state because of ImagePullBackOff.

The status ImagePullBackOff means that a container could not start because Kubernetes could not pull a container image for reasons such as:

- Invalid image name or
- Pulling from a private registry without imagePullSecret.

The BackOff part indicates that Kubernetes will keep trying to pull the image, with an increasing back-off delay.

Kubernetes raises the delay between each attempt until it reaches a compiled-in limit, which is 300 seconds (5 minutes).

**Pod State: CrashLoopBackOff**

When you see "CrashLoopBackOff," it means that kubelet is trying to run the container, but it keeps failing and crashing. After crashing, Kubernetes tries to restart the container automatically, but if the container keeps failing repeatedly, you end up in a loop of crashes and restarts, thus the term "CrashLoopBackOff." [reconstructed]

### 🛠️ Practical — Debugging Commands

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

---

## 6. Environment Variables (ENV & ENVFROM)

### What Interviewers Expect

A strong DevOps engineer would say: _"Env variables pass runtime configuration into containers. Use `env` for one-off values directly in the spec, and `envFrom` to inject in bulk from a ConfigMap or Secret. This decouples configuration from the container image, which is core to the 12-factor app methodology."_

### Beginner Explanation

Apps need values like DB hosts, ports, and passwords. Hardcoding them in the image is bad — env vars let you change them without rebuilding. **Real-world example:** Same Docker image for a Java app runs in dev, qa, and prod — only the env vars differ.

### Faculty Notes Explanation:

**ENV VARIABLES:**

It is a way to pass configuration information to containers running within pods. To set Env vars it include the env or envFrom field in the configuration file.

**ENV:** Allows you to set environment variables for a container, specifying a value directly for each variable through CLI / Command prompt

**ENVFROM:** PASSING Variables FROM FILE — 2 Types, configmaps and secrets

---

## 7. ConfigMaps

### What Interviewers Expect

A strong DevOps engineer would say: _"ConfigMaps store non-confidential data as key-value pairs and inject it into pods via env vars, command-line args, or mounted files. The size limit is 1MB — for larger configs we mount volumes or use external stores. Important: ConfigMaps are NOT encrypted; sensitive data should use Secrets."_

### Beginner Explanation

ConfigMaps = external config files. Change config without rebuilding the image. **Real-world example:** Storing log level, feature flags, or third-party API URLs in a ConfigMap and mounting it into all backend pods.

### Faculty Notes Explanation:

**CONFIGMAPS**

It is used to store the data in key-value pair, files, or command-line arguments that can be used by pods, containers in cluster.

But the data should be non-confidential. It does not provide security and encryption

If you want to provide Enryption use Secrets in K8S.

Limit of the configmap is only 1MB

But if you want to store more than 1MB configmap data mount volume or use a separate database or a file service

### 🛠️ Practical — Example with ENV (the wrong way first)

**Step 1 — Create a MariaDB deployment:**

```bash
kubectl create deploy newdb --image=mariadb
```

**Step 2 — Check pods — it will crash:**

```bash
kubectl get pods
```

**Step 3 — Inspect the crash logs (replace pod name with yours):**

```bash
kubectl logs newdb-794dd57dbc-tr7s9
```

_It crashes because MariaDB requires `MYSQL_ROOT_PASSWORD`._

**Step 4 — Set env variable directly via CLI:**

```bash
kubectl set env deploy newdb MYSQL_ROOT_PASSWORD=root123456
```

**Step 5 — Pods now run:**

```bash
kubectl get pods
```

**Step 6 — Clean up:**

```bash
kubectl delete deploy newdb
```

### 🛠️ Practical — Now with ConfigMap (the right way)

**Step 1 — Create a variables file:**

```bash
vi vars
```

**Step 2 — Paste:**

```
MYSQL_ROOT_PASSWORD=root123456
MYSQL_USER=admin
```

**Step 3 — Create a configmap from the file:**

```bash
kubectl create cm dbvars --from-env-file=vars
```

**Step 4 — Verify:**

```bash
kubectl get cm
kubectl describe cm dbvars
```

_This shows the data in plain text — not safe for passwords. Use Secrets instead (next section)._

**Step 5 — Create deployment again:**

```bash
kubectl create deploy newdb --image=mariadb
```

**Step 6 — Inject the configmap into the deployment:**

```bash
kubectl set env deploy newdb --from=configmaps/dbvars
```

**Step 7 — Verify pods are running:**

```bash
kubectl get pods
```

**Step 8 — Clean up:**

```bash
kubectl delete deploy newdb
```

---

## 8. Secrets

### What Interviewers Expect

A strong DevOps engineer would say: _"Secrets store sensitive data like passwords, TLS certs, and registry credentials. They're base64-encoded by default (not encrypted), but can be encrypted at rest by enabling EncryptionConfiguration in etcd. Types include Generic, TLS, and docker-registry. Inject via env, envFrom, or volume mounts."_

### Beginner Explanation

Secrets = ConfigMaps for confidential stuff. **Real-world example:** Storing DB passwords or TLS keys so they don't appear in plain YAML files committed to git.

### Faculty Notes Explanation:

**SECRETS:**

SECRETS: To store sensitive data in an unencrypted format like passwords, ssh-keys etc it uses base64 encoded format
password=reyaz (now we can encode and decode the value)

**WHY:** if i dont want to expose the sensitive info so we use SECRETS

By default k8s will create some Secrets these are useful from me to create communicate inside the cluster used to communicate with one resource to another in cluster

These are system created secrets, we need not to delete

**TYPES:**

- **Generic:** creates secrets from files, dir, literal (direct values)
- **TLS:** Keys and certs
- **Docker Registry:** used to get private images by using the password

### 🛠️ Practical Walkthrough

**Step 1 — Try deploying without a secret (it will fail):**

```bash
kubectl create deploy newdb --image=mariadb
kubectl get pods
```

**Step 2 — Create a secret from the CLI (literal = direct value):**

```bash
kubectl create secret generic password --from-literal=ROOT_PASSWORD=reyaz123
```

**Step 3 — Or create from the same `vars` file used earlier:**

```bash
kubectl create secret generic my-secret --from-env-file=vars
```

**Step 4 — Verify:**

```bash
kubectl get secrets
kubectl describe secret my-secret
```

**Step 5 — Inject into the deployment (will fail — wrong prefix):**

```bash
kubectl set env deploy newdb --from=secrets/my-secret
kubectl get pods
```

**Step 6 — Inject with correct `MYSQL_` prefix:**

```bash
kubectl set env deploy newdb --from=secret/my-secret --prefix=MYSQL_
```

_Without the prefix, MariaDB doesn't recognize the var name._

### 🛠️ Practical — Decoding Secrets

**View the base64 encoded value:**

```bash
kubectl get secrets password -o yaml
```

**Decode it (option 1):**

```bash
echo -n "LKJSKFHJHi" | base64 -d
```

**Decode it (option 2):**

```bash
echo -n "LKJSKFHJHi" | base64 --decode
```

**Clean up:**

```bash
kubectl delete deploy newdb
```

---

## 9. Resource Quota & Resource Limits

### What Interviewers Expect

A strong DevOps engineer would say: _"By default, pods have no resource limits — a single runaway pod can starve the whole cluster. We use `requests` (guaranteed minimum) and `limits` (hard cap) on individual pods, and Resource Quotas to cap total CPU/memory/object counts per namespace. CPU is measured in cores (1000m = 1 CPU); memory is in bytes (Mi, Gi)."_

### Beginner Explanation

Without limits, a buggy pod can eat all the cluster's CPU/RAM and crash everything else. **Real-world example:** Setting a quota so the `dev` namespace can't exceed 10 CPU + 20Gi memory, protecting `prod`.

### Faculty Notes Explanation:

**Resource Quota**

- K8S cluster can be divided into namespaces
- By default the pods in K8s will run with no limitation of Memory and CPU
- WE need to give the limit for the pod
- IT can limit the objects that can be created in a namespace and total amount of resources
- pod schedular in master will check the worker nodes on cpu and memory and create a pod in it
- we can set limits to CPU, Memory and storage
- CPU is measrured on Cores and memory on Bytes
- 1CPU = 1000 milliCPUs

**Requests** = how much we want
**Limit** = how much max we want

limits can be given to pod and nodes also [reconstructed]

### 🛠️ Practical — Deployment with Resource Limits

**Step 1 — Create the manifest:**

```bash
vi deploy.yml
```

**Step 2 — Paste this (high limits — pod count will be reduced if namespace quota is small):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ib-deployment
  labels:
    app: bank
spec:
  replicas: 3
  selector:
    matchLabels:
      app: bank
  template:
    metadata:
      labels:
        app: bank
    spec:
      containers:
        - name: cont1
          image: reyadocker/internetbankingrepo:latest
          resources:
            limits:
              cpu: "1"
              memory: 512Mi
```

**Step 3 — Apply and check:**

```bash
kubectl create -f deploy.yml
kubectl get pods
```

_Only 1 pod may run despite replicas=3, due to namespace quota restrictions._

**Step 4 — Lower the limits to fit within quota — edit `deploy.yml`:**

```yaml
resources:
  limits:
    cpu: "0.3"
    memory: 300Mi
```

**Step 5 — Re-apply and verify:**

```bash
kubectl apply -f deploy.yml
kubectl get po
kubectl get quota
```

**Step 6 — Clean up:**

```bash
kubectl delete -f deploy.yml
```

---

## 10. Metrics Server

### What Interviewers Expect

A strong DevOps engineer would say: _"Metrics Server collects CPU/memory stats from kubelets every ~15 seconds and aggregates them in-memory. It powers `kubectl top`, the Kubernetes Dashboard, and most importantly, the Horizontal Pod Autoscaler. Without it, autoscaling has no signal to act on."_

### Beginner Explanation

Metrics Server is the cluster's monitoring agent. **Real-world example:** Before turning on autoscaling for a web app, install Metrics Server so HPA knows when CPU crosses 70%.

### Faculty Notes Explanation:

The Metrics Server provides the resource usage data displayed in the Kubernetes Dashboard.

**How Metrics Server Works:**

**Kubelets:** Each node in a Kubernetes cluster runs a kubelet that periodically collects resource usage statistics from the node and the containers running on it.

**Metrics Server:** The Metrics Server collects these metrics from the kubelets and stores them in memory, aggregating them to be accessed by other components (like the HPA).

**In Short:**

- This metric server in K8S will collect metrics information like cpu, ram etc for all pods and nodes in the cluster
- A single deployment that works on most clusters, collect metrics every 15 secs
- We can use `kubectl top po/no` to see the metrics

### 🛠️ Practical — Install & Use Metrics Server

**Step 1 — Try to view metrics (will not work without metric server):**

```bash
kubectl top po
kubectl top no
```

**Step 2 — Install Metrics Server:**

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml
```

**Step 3 — Wait ~30 seconds, then check metrics:**

```bash
kubectl top pods
kubectl top nodes
```

**Step 4 — To remove (use `delete` instead of `apply`):**

```bash
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml
```

---

## 11. Horizontal Pod Autoscaler (HPA)

### What Interviewers Expect

A strong DevOps engineer would say: _"HPA automatically scales pod replicas based on CPU, memory, or custom metrics. It depends on Metrics Server. We define min/max replicas and a target utilization — for example, scale between 2 and 10 pods, keeping average CPU at 70%."_

### Beginner Explanation

HPA = auto-scaling for pods based on real-time load. **Real-world example:** During a Black Friday sale, checkout-service pods auto-scale from 3 → 20 as CPU rises, then back to 3 once traffic drops.

### 🛠️ Practical — HPA Demo Deployment

**Step 1 — Create the manifest:**

```bash
vi auto.yml
```

**Step 2 — Paste:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mb-deployment
  labels:
    app: bank
spec:
  replicas: 3
  selector:
    matchLabels:
      app: bank
  template:
    metadata:
      labels:
        app: bank
    spec:
      containers:
        - name: cont1
          image: reyadocker/mobilebankingrepo:latest
```

**Step 3 — Apply:**

```bash
kubectl create -f auto.yml
```

**Step 4 — Verify pods and metrics:**

```bash
kubectl get pods
kubectl top pods
```

**Step 5 — Check HPA objects:**

```bash
kubectl get hpa
```

**Note:** If you want to delete the metric server, use `delete` instead of `apply`:

```bash
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml
```

---

## 12. Persistent Volume (PV) & Persistent Volume Claim (PVC)

### What Interviewers Expect

A strong DevOps engineer would say: _"Pods are ephemeral — when they die, their data dies. PVs provide durable storage independent of pod lifecycle. PV is the actual storage resource (AWS EBS, NFS, etc.); PVC is a pod's request for storage. This is what makes stateful workloads like databases possible on Kubernetes."_

### Beginner Explanation

- **Stateless** = no saved data (pod dies → data gone). Web frontend.
- **Stateful** = saved data (pod dies → data survives). Database.

**Real-world example:** Running MySQL on K8s with an AWS EBS volume attached via PV/PVC — DB tables remain even if the pod restarts.

### Faculty Notes Explanation:

**PV - Persistent Volume**

**Stateless:** if i delete pod data is lost, because data is stored locally on the pod and instance

**Stateful:** if i delete the pod data is persistent, because we can store the data in external storage like AWS EBS

Kubernetes Persistent Volumes (PVs) provide a way to manage durable storage for applications running in a Kubernetes cluster.

Unlike ephemeral storage tied to the lifecycle of a pod, Persistent Volumes exist independently of pods and remain intact even after pods are deleted.

### 🛠️ Practical — Create PV (AWS EBS)

**Step 1 — Create the PV manifest:**

```bash
vi pv.yml
```

**Step 2 — Paste this (use YOUR EBS volume ID):**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: vol-0771f0561f66408c9
    fsType: ext4
```

**Step 3 — Create and verify:**

```bash
kubectl create -f pv.yml
kubectl get pv
```

### 🛠️ Practical — Create PVC

**Step 1 — Create the PVC manifest:**

```bash
vi pvc.yml
```

**Step 2 — Paste:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

**Step 3 — Apply:**

```bash
kubectl apply -f pvc.yml
```

### 🛠️ Practical — Use PVC in a Deployment

**Step 1 — Create deployment manifest:**

```bash
vi deploy.yml
```

**Step 2 — Paste:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ib-deployment
  labels:
    app: bank
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bank
  template:
    metadata:
      labels:
        app: bank
    spec:
      containers:
        - name: cont1
          image: centos
          command: ["/bin/bash", "-c", "sleep 10000"]
          volumeMounts:
            - name: my-pv
              mountPath: "/tmp/persistent"
      volumes:
        - name: my-pv
          persistentVolumeClaim:
            claimName: my-pvc
```

**Step 3 — Apply and verify:**

```bash
kubectl create -f deploy.yml
kubectl get pods
```

### 🛠️ Practical — Test Persistence

**Step 1 — Exec into the container (replace pod ID):**

```bash
kubectl exec -it podid-dfgdkjjf -- /bin/bash
```

**Step 2 — Inside the container, create test data:**

```bash
cd /tmp
ls
cd persistent
touch file{1..5}
vi file1
```

_(Add text like `this is from pod-1 pv`, then save & exit vi.)_

**Step 3 — Exit the container:**

```bash
exit
```

**Step 4 — Delete the pod (replace ID):**

```bash
kubectl delete pods podid-rfgdfdjg
```

**Step 5 — A new pod is auto-created — verify:**

```bash
kubectl get pods
```

**Step 6 — Exec into the new pod and confirm data still exists:**

```bash
kubectl exec -it podid-dfdgh435 -- /bin/bash
cd /tmp/persistent
ls
exit
```

**Or in one shot:**

```bash
kubectl exec -it podid-dfdgh435 -- ls /tmp/persistent
```

✅ Data persists across pod deletions — this is **stateful**.

### 🛠️ Practical — Resize PV

> First increase the EBS volume size in AWS console to 25.

**Step 1 — Check current capacity (will show 10Gi):**

```bash
kubectl describe pv
```

**Step 2 — Edit `pv.yml`, change storage to `20Gi`:**

```bash
vi pv.yml
```

**Step 3 — Re-apply:**

```bash
kubectl apply -f pv.yml
kubectl describe pv
```

**Step 4 — Clean up everything:**

```bash
kubectl delete -f .
```

---

## 13. Multi-Container Pods

### What Interviewers Expect

A strong DevOps engineer would say: _"A Pod can host multiple tightly-coupled containers that share the same network namespace and storage volumes. This is the foundation for patterns like Sidecar, Adapter, and Ambassador. Containers within a pod communicate via localhost."_

### Beginner Explanation

Most pods have 1 container. But sometimes 2+ containers need to work together as a single unit. **Real-world example:** Nginx serving content + a busybox log streamer printing logs every 5 seconds, sharing the same network and storage.

### Faculty Notes Explanation:

**MULTI-CONTAINER POD / Container designing pattern** — Generally, 1 pod contains 1 container, but if you want 2 containers in 1 pod:

In Kubernetes, you can run multiple containers within a single pod.

This setup is often used when the containers need to closely collaborate, such as when one container serves as a primary application and another container provides supporting services like logging or monitoring.

### 🛠️ Practical — Single Pod with 2 Containers

**Step 1 — Create the manifest:**

```bash
vi multicont.yml
```

**Step 2 — Paste:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
    - name: logger-container
      image: busybox:latest
      command:
        [
          "sh",
          "-c",
          "while true; do echo Hello from the logger container; sleep 5; done",
        ]
```

**Step 3 — Create and verify:**

```bash
kubectl create -f multicont.yml
kubectl get pods
```

**Step 4 — View logs from a specific container (use `-c <container-name>`):**

```bash
kubectl logs multi-container-pod -c logger-container
kubectl logs multi-container-pod -c nginx-container
```

---

## 14. Sidecar Design Pattern (with Adapter, Ambassador, Init Container)

### What Interviewers Expect

A strong DevOps engineer would say: _"Sidecar is the most common multi-container pattern — a helper container that extends the main app without modifying it. It's used for log aggregation, network proxies, or data sync. Related patterns include Adapter (standardizes the main container's output), Ambassador (proxies external connections), and Init Containers (run setup tasks and exit before the main container starts)."_

### Beginner Explanation

| Pattern            | Purpose                                              |
| ------------------ | ---------------------------------------------------- |
| **Sidecar**        | Adds helper functionality (e.g., log shipper)        |
| **Adapter**        | Standardizes main container's output format          |
| **Ambassador**     | Connects main container to outside world via proxy   |
| **Init Container** | Runs a one-time setup, then exits before main starts |

**Real-world example:** A Java app writes logs to a shared volume; an Nginx sidecar exposes those logs over HTTP so monitoring tools (like Prometheus) can scrape them.

### Faculty Notes Explanation:

**SIDE CAR:**

It creates a helper container to main container.
main container will have application and helper container will do help for main container.

**Adapter Design Pattern:**
standardize the output pattern of main container.

**Ambassador Design Pattern:**
used to connect containers with the outside world

**Init Container:**
it initialize the first work and exits later.

A sidecar container is a secondary container that runs alongside the main application container within the same Kubernetes Pod. This design pattern allows you to extend or enhance the functionality of the main application without modifying its code. Common use cases include log aggregation, data synchronization, and proxying network traffic.

### 🛠️ Practical — Log Aggregation with a Sidecar

**Step 1 — Create manifest:**

```bash
vi sidecar.yml
```

**Step 2 — Paste:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: log-aggregator-pod
spec:
  containers:
    - name: app-container
      image: alpine
      command: ["/bin/sh", "-c"]
      args: ["while true; do date >> /var/log/app.log; sleep 5; done"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log
    - name: sidecar-container
      image: nginx
      volumeMounts:
        - name: shared-logs
          mountPath: /usr/share/nginx/html
  volumes:
    - name: shared-logs
      emptyDir: {}
```

**Manifest Explanation:**

| Component                  | Purpose                                                                     |
| -------------------------- | --------------------------------------------------------------------------- |
| **app-container**          | Alpine image writing the current date to `/var/log/app.log` every 5 seconds |
| **sidecar-container**      | Nginx serving the shared volume over HTTP at `/usr/share/nginx/html`        |
| **shared-logs (emptyDir)** | Temporary shared storage created when the pod starts, deleted when it stops |

**Step 3 — Apply and verify:**

```bash
kubectl apply -f sidecar.yml
kubectl get pods
kubectl describe pods
```

**Step 4 — Exec into the sidecar to verify shared logs:**

```bash
kubectl exec -it log-aggregator-pod -c sidecar-container -- sh
```

**Step 5 — Inside the sidecar, view the app's logs:**

```bash
cd /usr/share/nginx/html
cat app.log
```

✅ Logs written by app-container are visible from sidecar-container via the shared volume.

**Step 6 — Clean up:**

```bash
kubectl delete -f sidecar.yml
```

---

## 15. Ingress

### What Interviewers Expect

A strong DevOps engineer would say: _"Ingress provides intelligent HTTP/HTTPS routing into the cluster — supporting host-based routing, path-based routing, SSL termination, and load balancing. Unlike a LoadBalancer service which routes only on ports, Ingress routes on URLs. It requires an Ingress Controller (NGINX, Traefik, etc.) to function."_

### Beginner Explanation

A `LoadBalancer` service is dumb — it just forwards ports. **Ingress is smart** — it can route based on URL paths or hostnames.

**Real-world example:**

- `paytm.com/movies` → movies service
- `paytm.com/recharge` → recharge service
- All behind a single AWS ELB

### Faculty Notes Explanation:

**INGRESS**

Ingress is a service to expose application, but we already have cluster ip, node port and load balancer, let see

- Ingress helps to expose HTTP and HTTPS routes from outside of the Cluster
- Ingress supports Host based routing and path based routing
- ingress supports load balancing and SSL termination
- IT redirect the incoming requests to the right services based on the web url or path in the address
- ingress provides encryption feature and helps to balance the load of the applications

**Explain Host based and Path based**

- **Host Based Routing:** ex: boom.com, web.boom.com, admin.boom.com
- **Path based routing:** boom.com/hello, boom.com/admin, Paytm.com/movies, Paytm.com/recharge etc

but services like load balancer, cluster ip, node port etc donest have these features

General load balancer routes the traffic based on ports and cant handle URL based routing

### 🛠️ Practical — Step 1: Check & Install Ingress Controller

**Verify no ingress exists yet:**

```bash
kubectl get ing
```

**Install the NGINX Ingress Controller:**

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.0/deploy/static/provider/cloud/deploy.yaml
```

**Verify resources:**

```bash
kubectl get pods
kubectl get deploy
kubectl get svc
kubectl get ingress
kubectl get service
```

**If required, clean up old resources:**

```bash
kubectl delete svc internetbanking
kubectl delete svc mobilebanking
kubectl delete deploy internetbanking
```

### 🛠️ Practical — Step 2: Create httpd Deployment & Service

**Create manifest:**

```bash
vi httpd.yml
```

**Paste:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
        - name: httpd
          image: httpd
          env:
            - name: TITLE
              value: "APACHE APP2"
---
apiVersion: v1
kind: Service
metadata:
  name: httpd
spec:
  type: ClusterIP
  ports:
    - port: 80
  selector:
    app: httpd
```

### 🛠️ Practical — Step 3: Create nginx Deployment & Service

**Create manifest:**

```bash
vi nginx.yml
```

**Paste:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          env:
            - name: TITLE
              value: "NGINX APP1"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: ClusterIP
  ports:
    - port: 80
  selector:
    app: nginx
```

### 🛠️ Practical — Step 4: Create Ingress Resource

**Create manifest:**

```bash
vi ingress.yml
```

**Paste:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: k8s-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /nginx(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: nginx
                port:
                  number: 80
          - path: /httpd(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: httpd
                port:
                  number: 80
          - path: /(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: nginx
                port:
                  number: 80
```

### 🛠️ Practical — Step 5: Apply & Test

**Apply all manifests:**

```bash
kubectl apply -f httpd.yml
kubectl apply -f nginx.yml
kubectl apply -f ingress.yml
```

**Verify:**

```bash
kubectl get deploy
kubectl get svc
kubectl get pods
kubectl get ingress
```

**Get the ELB URL of the ingress controller:**

```bash
kubectl get services ingress-nginx-controller --namespace=ingress-nginx
```

> **Important:** In AWS, allow all traffic in the node security group.

**Test in browser:**

```
http://<elb-url>/nginx     → routes to nginx service
http://<elb-url>/httpd     → routes to httpd service
```

### 🛠️ Practical — Step 6: Update Images (Rolling Update)

**Edit `httpd.yml` — change image:**

```bash
vi httpd.yml
```

Change `image: httpd` → `image: trainerreyaz/wordcounter`

**Edit `nginx.yml`:**

```bash
vi nginx.yml
```

Change `image: nginx` → `image: trainerreyaz/ib-image:latest`

**Re-apply:**

```bash
kubectl apply -f httpd.yml
kubectl apply -f nginx.yml
kubectl get pods
```

**Test again in browser:**

```
http://<elb-url>/nginx
http://<elb-url>/httpd
```

**Cleanup ingress controller service:**

```bash
kubectl delete services ingress-nginx-controller --namespace=ingress-nginx
```

### 📘 Key Components Explanation

**Annotations:**

- `nginx.ingress.kubernetes.io/ssl-redirect: "false"` → Disables auto HTTP→HTTPS redirect.
- `nginx.ingress.kubernetes.io/use-regex: "true"` → Enables regex path matching.
- `nginx.ingress.kubernetes.io/rewrite-target: /$2` → Rewrites the matched URI before forwarding.

**Rules:**

- `/nginx(...)` → routed to `nginx` service.
- `/httpd(...)` → routed to `httpd` service.
- `/(.*)` (catch-all) → routed to `nginx` service.

**Path Matching:** `pathType: ImplementationSpecific` allows the Ingress controller to interpret regex patterns.

**Rewrite Target:** `/$2` strips the prefix (e.g., `/nginx/foo` → `/foo`) before sending to the backend.

---

## 16. Pod Scheduling — Node Selector, Node Affinity, Taints & Tolerations

### What Interviewers Expect

A strong DevOps engineer would say: _"Kubernetes gives us multiple mechanisms to control pod placement. NodeSelector is the simplest — schedule only on nodes with matching labels. Node Affinity is more expressive with required and preferred rules. Taints repel pods from nodes unless those pods have matching tolerations. Together, these enable workload isolation, GPU scheduling, and dedicated node pools for production."_

### Beginner Explanation

By default, the scheduler picks any healthy node. Sometimes you need control over _which_ node a pod lands on.

| Mechanism                | What it does                                               |
| ------------------------ | ---------------------------------------------------------- |
| **NodeSelector**         | Simple — schedule only on nodes with matching labels       |
| **Node Affinity**        | Advanced — supports preferred/required rules and operators |
| **Taints & Tolerations** | Repel pods from a node unless they tolerate the taint      |

**Real-world example:** Label GPU nodes with `gpu=true`. Use NodeSelector so ML training pods land only there. Add a taint so regular pods can't accidentally schedule onto GPU nodes.

### Faculty Notes Explanation:

# **PODS SCHEDULING**

In Kubernetes, Node Selector, Node Affinity, Taints and Tolerations are mechanisms that influence how Pods are scheduled onto Nodes within a cluster.

- Node Selector
- Node Affinity
- Taints and Tolerations

**1. Node Selector**

NodeSelector is the simplest form of node selection constraint, allowing Pods to be scheduled only on Nodes with specific labels. By specifying a NodeSelector in a Pod's specification, you can ensure that the Pod runs only on Nodes that match the given label criteria.

**NodeSelector:** Use when you have simple, specific constraints for Pod placement based on Node labels

In the below manifest file, we are creating 2 pods and those pods should be scheduled in ib-node labeled node.

### 🛠️ Practical — Node Selector

**Step 1 — Label a node (replace `<node-name>`):**

```bash
kubectl label nodes <node-name> node-type=ib-node
```

**Step 2 — Verify the label:**

```bash
kubectl get nodes --show-labels
```

**Step 3 — Create the manifest:**

```bash
vi nodeselector.yml
```

**Step 4 — Paste:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ib-deployment
  labels:
    app: bank
spec:
  replicas: 2
  selector:
    matchLabels:
      app: bank
  template:
    metadata:
      labels:
        app: bank
    spec:
      nodeSelector:
        node-type: ib-node
      containers:
        - name: cont1
          image: nginx
```

_[Manifest below `selector:` was cut off in source — reconstructed based on context.]_

**Step 5 — Apply:**

```bash
kubectl apply -f nodeselector.yml
```

**Step 6 — Verify pods scheduled on the labeled node:**

```bash
kubectl get pods -o wide
```

---

## 📋 Master Command Cheat Sheet

```bash
# ====== Namespaces ======
kubectl get ns
kubectl create ns <name>
kubectl config view
kubectl config set-context --current --namespace=<ns>

# ====== Pods ======
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container-name> -- sh
kubectl delete pod <pod-name>
kubectl run <name> --image=<image>

# ====== Deployments ======
kubectl create deploy <name> --image=<image>
kubectl get deploy
kubectl set env deploy <name> KEY=VALUE
kubectl set env deploy <name> --from=configmaps/<cm>
kubectl set env deploy <name> --from=secrets/<secret> --prefix=PREFIX_
kubectl delete deploy <name>

# ====== ConfigMaps ======
kubectl create cm <name> --from-env-file=<file>
kubectl create cm <name> --from-literal=KEY=VALUE
kubectl get cm
kubectl describe cm <name>

# ====== Secrets ======
kubectl create secret generic <name> --from-literal=KEY=VALUE
kubectl create secret generic <name> --from-env-file=<file>
kubectl get secrets
kubectl describe secret <name>
kubectl get secrets <name> -o yaml
echo -n "<base64-string>" | base64 -d

# ====== Services & Ingress ======
kubectl get svc
kubectl describe svc <name>
kubectl get ingress
kubectl get services ingress-nginx-controller --namespace=ingress-nginx

# ====== Persistent Volumes ======
kubectl get pv
kubectl get pvc
kubectl describe pv

# ====== Monitoring ======
kubectl top pods
kubectl top nodes
kubectl get hpa
kubectl get quota

# ====== Nodes & Labels ======
kubectl get nodes
kubectl get nodes --show-labels
kubectl label nodes <node-name> KEY=VALUE

# ====== Manifests ======
kubectl apply -f <file.yml>      # create or update
kubectl create -f <file.yml>     # create new only
kubectl delete -f <file.yml>     # delete from file
kubectl delete -f .              # delete everything in current dir
```

---

## 🎯 Final Interview Tips

1. **Always mention `kubectl describe` and `kubectl logs` when asked about debugging** — these are the bread and butter.
2. **Know the difference between `apply` and `create`** — `apply` is idempotent (create + update), `create` is one-time.
3. **Understand the relationship: Deployment → ReplicaSet → Pod → Container.**
4. **Be ready to explain why something is in `Pending`, `CrashLoopBackOff`, or `ImagePullBackOff` state.**
5. **For stateful workloads, always discuss PV/PVC and StorageClass.**
6. **For production traffic, explain Ingress + Ingress Controller + TLS termination.**
