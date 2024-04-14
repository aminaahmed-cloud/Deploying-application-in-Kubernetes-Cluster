# Deploying Application in Kubernetes Cluster

This guide provides step-by-step instructions for setting up a complete application with MongoDB and Mongo Express in a Kubernetes cluster.

## Overview of Kubernetes Components

Before starting with the deployment, it's essential to understand the key components of Kubernetes, including Pods, Deployments, Services, ConfigMaps, and Secrets.

## Step 1: Create MongoDB Deployment and Internal Service

Create a MongoDB Deployment to manage MongoDB instances and an internal Service to allow communication between different parts of the application.

```yaml
# mongo.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: "your_username_here"  # Replace "your_username_here" with your desired username
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: "your_password_here"  # Replace "your_password_here" with your desired password

```

<img src="https://i.imgur.com/AbwbEm5.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/NmmXkQB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


## Step 2: Create Secret Configuration

Create a Secret to store sensitive information like database credentials.

```sh
% echo -n 'username' | base64
% echo -n 'password' | base64
```

```yaml
# mongo-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  mongo-root-username: dXNlcm5hbWU=
  mongo-root-password: cGFzc3dvcmQ=
```

<img src="https://i.imgur.com/ZeWwEXR.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


**- Apply the secret configuration:**

```bash
% kubectl apply -f mongo-secret.yaml
```

**- Update the MongoDB Deployment with the secret configuration.**

Apply the changes:

```bash
% kubectl apply -f mongo.yaml
```

<img src="https://i.imgur.com/PWwqrjL.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


## Step 3: Create MongoDB External Service

Create an external Service to expose MongoDB externally.

**- Combine Deployment and Service configurations in a single file:**

```yaml
# mongo.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
  - protocol: TCP
    port: 27017
    targetPort: 27017
```

<img src="https://i.imgur.com/4nbxGSo.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

Apply the changes:

```bash
% kubectl apply -f mongo.yaml
```

<img src="https://i.imgur.com/fbMwSHd.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


## Step 4: Create Mongo Express Deployment, Service, and ConfigMap

**- Create a ConfigMap to store MongoDB connection details and configure Mongo Express.**

```yaml
# mongo-configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service
```


<img src="https://i.imgur.com/bvtuIYZ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

**- Create a Mongo Express Deployment:**

```yaml
# mongo-express.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom:
            configMapKeyRef:
              name: mongodb-configmap
              key: database_url
```

<img src="https://i.imgur.com/7c7ZGa2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

<img src="https://i.imgur.com/sip34Me.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

<img src="https://i.imgur.com/nsIJe2X.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


---

**- Apply the ConfigMap and Mongo Express Deployment:**

```bash
% kubectl apply -f mongo-express.yaml
```
## Step 5: Access the external service:

```bash
% minikube service mongo-express-service
```

<img src="https://i.imgur.com/LF1BZnU.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---
