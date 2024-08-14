# Kubernetes Ingress Controller on Minikube Cluster



### In this demo, we will see how to use ingress controller to route the traffic on different services.



### Pre-requisites to implement this project:



- Create 1 virtual machine on AWS with 2 CPU, 4GB of RAM (t2.medium)
- Setup minikube on it [Minikube setup](https://github.com/rajatchauhan-git/kubernetes_zero_to_hero/blob/master/Day%201/README.md).

# 

### What we are going to implement:



- In this demo, we will create two deployment and services i.e nginx and apache and with the help of ingress, we will route the traffic between the services

# 

## Steps to implement ingress:



**1) Create minikube cluster as mentioned in pre-requisites :**

# 

**2) Check minikube cluster status and nodes :**

```
minikube status
kubectl get nodes
```



# 

**3) Create one yaml file for apache deployment and service :**

```
# apache-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: apache
        image: httpd:2.4
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: apache-service
spec:
  selector:
    app: apache
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```



# 

**4) Apply apache deployment :**

```
kubectl apply -f apache-deployment.yaml
```



# 

**5) Create one more yaml file for nginx deployment and service :**

```
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```



# 

**6) Apply nginx deployment :**

```
kubectl apply -f nginx-deployment.yaml
```



# 

**7) Enable the Ingress Controller :**

```
minikube addons enable ingress
```



# 

**8) Now create an Ingress resource that routes traffic to the Apache and NGINX services based on the URL path.**

```
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apache-nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: "rcjerry.com"
    http:
      paths:
      - path: /apache
        pathType: Prefix
        backend:
          service:
            name: apache-service
            port:
              number: 80
      - path: /nginx
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```



# 

**9) Apply the Ingress resource :**

```
kubectl apply -f ingress.yaml
```



# 

**10) To test the Ingress, map the hostname to the Minikube IP in your \*/etc/hosts\* file :**

```
echo "$(minikube ip) rcjerry.com" | sudo tee -a /etc/hosts
```



OR Open **/etc/hosts** file and add your minikube ip and domain name at the last.

# 

**11) Now, test the routing :**

- curl http://rcjerry.com/apache to access the Apache service.

```
curl http://rcjerry.com/apache
```



- curl http://rcjerry.com/nginx to access the NGINX service.

```
curl http://rcjerry.com/nginx
```



OR

- port forward to access the Apache service on browser.

  ```
  kubectl port-forward svc/apache-service 8081:80 --address 0.0.0.0 &
  ```

  

- port forward to access the NGINX service on browser.

  ```
  kubectl port-forward svc/nginx-service 8082:80 --address 0.0.0.0 &
  ```

------

<u>[Original Doc](https://github.com/LondheShubham153/kubestarter/tree/main/Ingress)</u>

