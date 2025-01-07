# WebApp-Deployment-with-MongoDB-on-Kubernetes
Deploying a webapp with Mongo DB using Kubernetes

A realistic application setup using kubernetes cluster

Deploying a mongo DB database and a webapplication which will connect to the mongo db database using external configuration data from configmap and secret

Making the webapplication accessible externally though the browser

We are going to create 4 kubernetes configfile:

1. Config map - MongoDB Endpoint
2. Secret - MongoDB User and Password
3. Deployment/Service - MongoDB Application with Internal Service
4. Deployment/Service - Our Own web app with external service



