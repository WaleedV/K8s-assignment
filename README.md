# Deploying  Web Application  and MongoDB on  Kubernetes Cluster

This guide will walk you through the process of deploying a web application and a MongoDB database on a Kubernetes cluster. The web application will connect to the MongoDB database using external configuration data from a ConfigMap and a Secret. Finally, the web application will be accessible externally from the browser.


# Prerequisites

Before you begin, make sure you have the following:

-   A Kubernetes cluster up and running
-   `kubectl`  command-line tool installed and configured to connect to your Kubernetes cluster
-   Docker  installed on your local machine to build and push Docker images to a  container registry

##  Step 1: Create ConfigMap and Secret

>Create a ConfigMap and a Secret to store the external configuration data that will be used by the web application.
```yaml
#mongo-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
	name: mongo-config
data:
	mongo-url: mongo-service
```
```yaml
 #mongo-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque
data:
  DB_USER: <base64-encoded-username>
  DB_PASSWORD: <base64-encoded-password>
 ```
 >Replace `<base64-encoded-username>` and `<base64-encoded-password>` with your MongoDB username and password, encoded in base64.


### Create the ConfigMap and Secret using the following commands:

```bash 
kubectl apply -f mongo-config.yaml
kubectl apply -f mongo-secret.yaml
```

##  Step 2: Deploy MongoDB
Create a deployment and a service for MongoDB using the following YAML file:
```yaml
#mongo.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
  labels:
    app: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongodb
        image: mongo:5.0
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-user
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-password  
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  selector:
    app: mongo
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```
>This YAML file creates a deployment with a single replica of MongoDB, and a service that exposes port 27017 for other services to connect to.
### Deploy MongoDB using the following command:
```bash 
kubectl apply -f mongo.yaml
```
## Step 3: Deploy Web Application
Create a deployment and a service for your web application using the following YAML file:
````yaml
#webapp.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nanajanashia/k8s-demo-app:v1.0
        ports:
        - containerPort: 3000
        env:
        - name: USER_NAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-user
        - name: USER_PWD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-password 
        - name: DB_URL
          valueFrom:
            configMapKeyRef:
              name: mongo-config
              key: mongo-url
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 30100
 ````
 >This YAML file creates a deployment with a single replica of your web application, and a service that exposes port 3000 for external access.
 ### Deploy your web application using the following command:
 ```bash
 kubectl apply -f webapp.yaml
 ```
 

##  Step 4: Access Web Application

To access your web application, get the external IP address of the `webapp-service` service using the following command:
```bash
kubectl get service webapp-service
```
>Open a browser and navigate to  `http://<external-ip>`  to access your web application.

Congratulations, you have successfully deployed a web application and a MongoDB database on a Kubernetes cluster!
