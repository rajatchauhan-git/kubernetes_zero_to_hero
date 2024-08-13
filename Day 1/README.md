This guide provides step-by-step instructions for installing Minikube on Ubuntu. Minikube allows you to run a single-node Kubernetes cluster locally for development and testing purposes.

## Pre-requisites

- Ubuntu OS
- sudo privileges
- Internet access
- Virtualization support enabled (Check with `egrep -c '(vmx|svm)' /proc/cpuinfo`, 0=disabled 1=enabled)

------

### Step 1: Update System Packages

Update your package lists to make sure you are getting the latest version and dependencies.

Copy

Copy

```yaml
sudo apt update
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723507879924/d5a3282a-8981-4bb1-92b5-1e8defd0a568.png?auto=compress,format&format=webp)

### Step 2: Install Required Packages

Install some basic required packages.

Copy

Copy

```yaml
sudo apt install -y curl wget apt-transport-https
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723507948126/1ae09a1b-d90c-4765-9b52-037a55b87c81.png?auto=compress,format&format=webp)

------

### Step 3: Install Docker

Minikube can run a Kubernetes cluster either in a VM or locally via Docker. This guide demonstrates the Docker method.

Copy

Copy

```yaml
sudo apt install -y docker.io
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723507977655/c3e23a31-b2ca-4fdb-bdd2-b84d63b2bbf4.png?auto=compress,format&format=webp)

Start and enable Docker.

Copy

Copy

```yaml
sudo systemctl enable --now docker
```

Add current user to docker group (To use docker without root)

Copy

Copy

```yaml
sudo usermod -aG docker $USER && newgrp docker
```

Now, logout (use `exit` command) and connect again.

------

### Step 4: Install Minikube

First, download the Minikube binary using `curl`:

Copy

Copy

```yaml
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723508030927/d974a788-8044-45a8-ab73-534a8ab34d8f.png?auto=compress,format&format=webp)

Make it executable and move it into your path:

Copy

Copy

```yaml
chmod +x minikube
sudo mv minikube /usr/local/bin/
```

------

### Step 5: Install kubectl

Download kubectl, which is a Kubernetes command-line tool.

Copy

Copy

```yaml
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723508086502/2ab0ca76-6919-4dc9-a1d7-301f7ce95e6b.png?auto=compress,format&format=webp)

**Check the above image ⬆️ Make it executable and move it into your path:**

Copy

Copy

```yaml
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

------

### Step 6: Start Minikube

Now, you can start Minikube with the following command:

Copy

Copy

```yaml
minikube start --driver=docker --vm=true
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723508374217/e10318c2-a6e2-4726-b4d8-d3fee54e90bc.png?auto=compress,format&format=webp)

This command will start a single-node Kubernetes cluster inside a Docker container.

------

### Step 7: Check Cluster Status

Check the cluster status with:

Copy

Copy

```yaml
minikube status
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723508400459/cb6e303f-a0cb-4f64-a185-08ee01cb429f.png?auto=compress,format&format=webp)

You can also use `kubectl` to interact with your cluster:

Copy

Copy

```yaml
kubectl get nodes
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723508419796/2b0fd8c9-7bd0-449e-bbed-5284a92d2624.png?auto=compress,format&format=webp)

------

### Step 8: Stop Minikube

When you are done, you can stop the Minikube cluster with:

Copy

Copy

```yaml
minikube stop
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723508454905/9ff3a9b0-1cf8-4958-b123-768f3f492136.png?auto=compress,format&format=webp)

------

### Delete Minikube Cluster (Optional)

If you wish to delete the Minikube cluster entirely, you can do so with:

Copy

Copy

```yaml
minikube delete
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723508479450/f11787c8-9f7a-40b3-bc95-7c41e2f58f05.png?auto=compress,format&format=webp)