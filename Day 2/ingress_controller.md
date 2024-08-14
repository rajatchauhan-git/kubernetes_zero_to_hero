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

![image-20240814134802007](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814134802007.png)

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

![image-20240814135757880](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814135757880.png)

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

![image-20240814135825279](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814135825279.png)

# 

**7) Enable the Ingress Controller :**

```
minikube addons enable ingress
```

![image-20240814135907827](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814135907827.png)

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

![image-20240814135955422](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814135955422.png)

# 

**10) To test the Ingress, map the hostname to the Minikube IP in your \*/etc/hosts\* file :**

```
echo "$(minikube ip) rcjerry.com" | sudo tee -a /etc/hosts
```

![image-20240814140027836](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814140027836.png)

OR Open **/etc/hosts** file and add your minikube ip and domain name at the last.

# 

**11) Now, test the routing :**

- curl http://rcjerry.com/apache to access the Apache service.

```
curl http://rcjerry.com/apache
```

![image-20240814140127912](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814140127912.png)



- curl http://rcjerry.com/nginx to access the NGINX service.

```
curl http://rcjerry.com/nginx
```

![image-20240814140213159](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814140213159.png)



OR

`*Note: Before running the port forwarding commands you need to enable port 8081 & 8082 from inbound rules in your instance to access the APACHE AND NGINIX SERVICE on browser*`

**Step1:** Select the Instance from the AWS Console and got to security tab.

![image-20240814140957666](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814140957666.png)

**Step2:** Click on the security group and edit inbound rules



![image-20240814141128278](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814141128278.png)



**Step3:** Add port 8081 and 8082 from Add rule.



![image-20240814141242731](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814141242731.png)





- port forward to access the Apache service on browser.

  ```
  kubectl port-forward svc/apache-service 8081:80 --address 0.0.0.0 &
  ```

  ![image-20240814140411612](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814140411612.png)
  
  

- port forward to access the NGINX service on browser.

  ```
  kubectl port-forward svc/nginx-service 8082:80 --address 0.0.0.0 &
  ```

![image-20240814141516991](C:\Users\chumc\AppData\Roaming\Typora\typora-user-images\image-20240814141516991.png)

------

<u>[Original Doc](https://github.com/LondheShubham153/kubestarter/tree/main/Ingress)</u>

