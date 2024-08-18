![Kubernetes end-to-end project with deployment of Django notes application](https://cdn.hashnode.com/res/hashnode/image/upload/v1723996753093/7b4a4969-fb3c-40d4-a915-292e2dcdfea0.png?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp)



This project extends the deployment of a Django Notes application, initially containerized and pushed to DockerHub, by deploying and managing it on a Kubernetes cluster using Minikube. The application will be deployed using the `chauhanrajat/note-app-jenkins:latest` Docker image within a dedicated Kubernetes namespace, `notes-app`, ensuring a robust, scalable, and secure environment.

------

### Minikube Cluster Installation

Before proceeding, ensure that Minikube is installed on your system. For detailed steps, refer to the [Minikube Installation Guide](https://chauhanrajatwork.hashnode.dev/kubernetes-architecture-and-components-kubernetes-installation-and-configuration).

------

### Step 1: Namespace Creation

To organize Kubernetes resources, create a dedicated namespace named `notes-app`.

**namespace.yml:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: notes-app
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723996964928/bd27a755-bf9a-4c75-9360-5ef6f697fceb.png?auto=compress,format&format=webp)

Apply the namespace configuration:



```yaml
kubectl apply -f namespace.yml
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723996994995/86d4be5e-c2f5-4f31-a53a-69108c030cfa.png?auto=compress,format&format=webp)

### Step 2: Deployment Creation

Deploy the `notes-app` using a Deployment resource to manage replicas and application updates.

**deployment.yml:**



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notes-app-deployment
  namespace: notes-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: notes-app
  template:
    metadata:
      labels:
        app: notes-app
    spec:
      containers:
      - name: notes-app
        image: chauhanrajat/note-app-jenkins:latest
        ports:
        - containerPort: 8000
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723997062653/6f133702-1732-4f4a-981c-ff39894c9232.png?auto=compress,format&format=webp)

Apply the deployment configuration:

```yaml
kubectl apply -f deployment.yml
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723997102946/a70d8dd9-9c1b-4c77-81f2-70c48c03d22a.png?auto=compress,format&format=webp)

### Step 3: Service Creation

Expose the `notes-app` deployment using a Kubernetes Service to enable external access.

**service.yml:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: notes-app-service
  namespace: notes-app
spec:
  selector:
    app: notes-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723997151021/74e57096-56ee-47b6-820d-4adb1d5eefe0.png?auto=compress,format&format=webp)

Apply the service configuration:



```yaml
kubectl apply -f service.yml
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723997173344/6efc5f54-7ee4-48c3-8900-3a2ee01b3dd1.png?auto=compress,format&format=webp)

For testing, use port forwarding to access the application:



```yaml
kubectl port-forward service/notes-app-service 8000:80 --address=0.0.0.0 -n notes-app
```

- For testing, port forwarding of traffic.

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723998321500/951b5967-2c4a-4577-bf00-4e20214eb753.png?auto=compress,format&format=webp)

- Copy the Public IP of the instance with port 8000 and access the application on the browser.

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999437822/3eed8857-cff1-4bbc-be64-0086842d6de5.png?auto=compress,format&format=webp)

### Step 4: Persistent Volumes and Claims

Set up Persistent Volumes (PV) and Persistent Volume Claims (PVC) to handle persistent storage for the application.

**pv.yml:**



```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: notes-app-pv
  namespace: notes-app
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723998385221/576b3f5c-aa3c-4c15-80e7-5f10b1b15e32.png?auto=compress,format&format=webp)

Apply the PersistentVolume configuration:



```yaml
kubectl apply -f pv.yml
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723998418745/3d66998f-b1db-4fa4-a91b-de654ec9b7ec.png?auto=compress,format&format=webp)

**pvc.yml:**



```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: notes-app-pvc
  namespace: notes-app
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723998505377/3999a651-eb5d-4069-98ff-8218ebf0ed43.png?auto=compress,format&format=webp)

Apply the PersistentVolumeClaim configuration:



```yaml
kubectl apply -f pvc.yml
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723998535619/17f4c05b-5605-4b55-9afa-a2bb0c93b6c7.png?auto=compress,format&format=webp)

Update the deployment to attach the persistent storage:



```yaml
iapiVersion: apps/v1
kind: Deployment
metadata:
  name: notes-app-deployment
  namespace: notes-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: notes-app
  template:
    metadata:
      labels:
        app: notes-app
    spec:
      containers:
      - name: notes-app
        image: chauhanrajat/note-app-jenkins:latest
        ports:
        - containerPort: 8000
        volumeMounts:
        - name: notes-app-storage
          mountPath: /data  # Mount path inside the container
      volumes:
      - name: notes-app-storage
        persistentVolumeClaim:
          claimName: notes-app-pvc  # Reference to the PVC
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723998741282/2a12c8d7-fd67-44d1-9227-eb73d8959caf.png?auto=compress,format&format=webp)

Apply the deployment configuration:



```yaml
kubectl apply -f deployment.yml
```

### Step 5: Ingress Controller

Enable the Ingress addon in Minikube for path-based routing:



```yaml
minikube addons enable ingress
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723998841295/ad6376dd-ce9f-4f8b-9ccb-beef00cd5b7e.png?auto=compress,format&format=webp)

Set up an Ingress resource for the application:

**ingress.yml:**



```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: notes-app-ingress
  namespace: notes-app
spec:
  rules:
  - host: notes-app
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: notes-app-service
            port:
              number: 80
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: notes-app-service
            port:
              number: 80
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999026403/6d76ebef-6357-44b6-86c8-c5b862081a7d.png?auto=compress,format&format=webp)

Apply the Ingress configuration:



```yaml
kubectl apply -f ingress.yml
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999043700/fd5bf596-dac7-4413-9ce1-3b977c0fd497.png?auto=compress,format&format=webp)

Add a domain entry in `/etc/hosts`:



```yaml
sudo nano /etc/hosts
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999328269/b648bd5b-ca0e-4536-a344-7592db016564.png?auto=compress,format&format=webp)

Test the application by sending a query to the URL:



```yaml
curl notes-app
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999291750/ac56a12d-a36a-4cf8-ac2b-e7ac8b603be6.png?auto=compress,format&format=webp)

- This is what we called it as path-based routing using the Ingress controller.

### Step 6: Network Policies and CNI

Define Network Policies to control the network traffic between pods.

**networkPolicy.yml:**



```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: notes-app-network-policy
  namespace: notes-app
spec:
  podSelector:
    matchLabels:
      app: notes-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: notes-app
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: notes-app
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999624635/a5448b0a-59f2-44fa-99be-79563f28c1b7.png?auto=compress,format&format=webp)

Apply the NetworkPolicy configuration:



```yaml
kubectl apply -f networkPolicy.yml
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999643281/1a385c67-609d-4967-98b2-d4bea2a6892f.png?auto=compress,format&format=webp)

### Step 7: Job and CronJob

Define a Job for one-time tasks and a CronJob for scheduled tasks.

**job.yml:**



```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: notes-app-job
  namespace: notes-app
spec:
  template:
    spec:
      containers:
      - name: notes-app-job
        image: chauhanrajat/note-app-jenkins:latest
        ports:
        - containerPort: 8000
      restartPolicy: OnFailure
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999688135/2f0d2dfa-5706-4a3c-9ebd-7746128de6b1.png?auto=compress,format&format=webp)

**cron.yml:**



```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: notes-app-cronjob
  namespace: notes-app
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: notes-app-cronjob
            image: chauhanrajat/note-app-jenkins:latest
            ports:
            - containerPort: 8000
          restartPolicy: OnFailure
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999745777/8e0c77a8-9655-4fd4-97a2-46e938d9345e.png?auto=compress,format&format=webp)

Apply the configurations:



```yaml
kubectl apply -f job.yml
kubectl apply -f cron.yml
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999767000/abea3558-2054-4325-9f62-78a1f0bc9f4b.png?auto=compress,format&format=webp)

### Step 8: Horizontal Pod Autoscaler (HPA)

Enable the `metrics-server` addon in Minikube for HPA:

```yaml
minikube addons enable metrics-server
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999812438/285f2ed9-b75a-4de3-9902-40571216669f.png?auto=compress,format&format=webp)

Modify the deployment to include resource limits:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notes-app-deployment
  namespace: notes-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: notes-app
  template:
    metadata:
      labels:
        app: notes-app
    spec:
      containers:
      - name: notes-app
        image: chauhanrajat/note-app-jenkins:latest
        ports:
        - containerPort: 8000
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "200m"
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999880919/2b6aeeca-6844-43b8-9931-f3e42039df39.png?auto=compress,format&format=webp)

Implement HPA:

**hpa.yml:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: notes-app-hpa
  namespace: notes-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: notes-app-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 15
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999924581/4e6fa6ad-3a79-4de5-99dd-7ffefc0bca84.png?auto=compress,format&format=webp)

Apply the HPA configuration:

```yaml
kubectl apply -f hpa.yml
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999946288/be9dcbdc-979c-4e9e-b645-0e5d340491cc.png?auto=compress,format&format=webp)

Monitor CPU utilization and autoscaling:

```yaml
watch kubectl get hpa -n notes-app
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1724000001940/1ef244aa-4d7d-4c9a-895c-187fedef428b.png?auto=compress,format&format=webp)

### Step 9: Role-Based Access Control (RBAC)

Create a service account:

```yaml
kubectl create serviceaccount k8s-user
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1724000023675/386c7b59-239c-4204-9343-993ae13bfe86.png?auto=compress,format&format=webp)

Define a Role and RoleBinding for RBAC:

**role.yml:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: notes-app-role
  namespace: notes-app
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1724000063306/e6d1b904-2dc2-4056-bf27-41a35b4e6567.png?auto=compress,format&format=webp)

**roleBinding.yml:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: notes-app-rolebinding
  namespace: notes-app
subjects:
- kind: User
  name: k8s-user # Replace with your Kubernetes user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: notes-app-role
  apiGroup: rbac.authorization.k8s.io
```

- Role defines permissions, and RoleBinding assigns those permissions to a user or set of users.
- Testing of RBAC configurations,

- Run commands using --as=user flag (Where `user` **is the name of the user you wish to impersonate**.)
- As `k8s-user` has to get permission,

1. ### **Step 10: Testing application on the browser**

- Forward port on containerPort from the local port and try to access the application,



```yaml
kubectl port-forward service/notes-app-service 8000:80 --address=0.0.0.0 -n notes-app
```

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1724001239356/30a08cfa-2463-458a-b329-9aa0a1580ca5.png?auto=compress,format&format=webp)

![img](https://cdn.hashnode.com/res/hashnode/image/upload/v1723999437822/3eed8857-cff1-4bbc-be64-0086842d6de5.png)

------

```
💡If you need help or have any questions, just leave them in the comments! 📝 I would be happy to answer them!
```

```
💡If you found this post useful, please give it a thumbs up 👍 and consider following for more helpful content. 😊
```

### Thank you for taking the time to read! 💚

