# WebApp-Deployment-with-MongoDB-on-Kubernetes
This repository demonstrates a Deployment of a webapp with Mongo DB using Kubernetes. A realistic application setup using kubernetes cluster. Deploying a mongo DB database and a webapplication which will connect to the mongo db database using external configuration data from configmap and secret

Making the webapplication accessible externally though the browser

![Arc](https://github.com/Kenneth7117/WebApp-Deployment-with-MongoDB-on-Kubernetes/blob/main/Architecture.png)

## We are going to create 4 kubernetes configfile:

1. Config map - MongoDB Endpoint
2. Secret - MongoDB User and Password
3. Deployment/Service - MongoDB Application with Internal Service
4. Deployment/Service - Our Own web app with external service


## WorkFlow:

### 1. Config map - MongoDB Endpoint

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-config
data:
  mongo-url: mongo-service
```
- This created a key value pair to map the MongoDB with the Webapp 

### 2. Secret - MongoDB User and Password

```
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque
data:
  mongo-user: bW9uZ291c2Vy
  mongo-password: bW9uZ29wYXNzd29yZA==
```
- This created a key value pair of the MongoDB Credentials with the MongoDB pod.
- To be noted Kubernetes needs secrets data added with base64 encoding. Hence the user and password are encoded as such
  
### 3. Deployment/Service - MongoDB Application with Internal Service

This YAML file deploys a MongoDB instance and exposes it via a service for communication within the Kubernetes cluster.

```
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
- The apiVersion: apps/v1 and kind: Deployment specify that this YAML defines a Kubernetes Deployment for managing MongoDB pods.
- The metadata section provides a name (mongo-deployment) and labels (app: mongo) to identify the deployment.
- The spec.replicas: 1 ensures that only one pod will be running as part of this deployment.
- The selector.matchLabels ensures the deployment manages pods with the label app: mongo.
- The template defines the structure of the pods, including metadata (labels: app: mongo) and specifications.
- Under spec.containers, the pod runs a single container. Port: Exposes port 27017 inside the container.
- Environment variables for MongoDB initialization are dynamically set using Kubernetes Secrets.MONGO_INITDB_ROOT_USERNAME and MONGO_INITDB_ROOT_PASSWORD retrieve their values from the mongo-secret secret.
- The second part of the YAML defines a Kubernetes Service (apiVersion: v1, kind: Service) named mongo-service.
- The service targets pods labeled with app: mongo, ensuring it connects to the MongoDB pod.
- The service listens on port 27017 and forwards traffic to the same port on the pod (targetPort: 27017), using the TCP protocol.

### 4. Deployment/Service - Our Own web app with external service

This YAML file deploys a web application containerized with Docker, sets up secure and configurable environment variables, and exposes it to external users through a NodePort service.

```
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
```
- The apiVersion: apps/v1 and kind: Deployment specify the creation of a Deployment for the webapp application.
- The metadata section names the Deployment webapp-deployment and assigns it the label app: webapp for identification.
- The selector.matchLabels binds this Deployment to manage pods with the label app: webapp.
- Port: Exposes port 3000 inside the container.
- The container uses environment variables. USER_NAME and USER_PWD are retrieved from a Kubernetes secret named mongo-secret. DB_URL is retrieved from a ConfigMap named mongo-config.
- The apiVersion: v1 and kind: Service define a Service named webapp-service to expose the web application.
- The Service is of type NodePort, allowing external access to the application through a specific port on the node.
- The Service, Listens on port 3000 (cluster-internal). Forwards traffic to port 3000 in the pod. Exposes the application externally on node port 30100.

### 5. Hosting the App in Kubernetes
```
apply -f Mongo-config.yaml
apply -f Mongo-secret.yaml
apply -f Mongo.yaml
apply -f Webapp.yaml
```
- These above commands apply and deploy the configurations to the Kubernetes cluster

```
kubectl get all
minikube ip
```
- Can be used to validate the cluster
- Get the IP of the node to access the hosted application

## App Output:

Accessing the ip with the configured External port we can access the deployed Kubernetes application

![Arc](https://github.com/Kenneth7117/WebApp-Deployment-with-MongoDB-on-Kubernetes/blob/main/Output.png)
