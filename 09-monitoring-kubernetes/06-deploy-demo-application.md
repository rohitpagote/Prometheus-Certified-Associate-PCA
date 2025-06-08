# Deploy Demo Application
- In this guide, we'll see how to configure Prometheus - installed via the Prometheus Operator - to monitor a `Node.js` application running in a Kubernetes cluster. 
- In addition to tracking the Kubernetes infrastructure, Prometheus will also scrape metrics directly from our application by targeting the `/swagger-stats/metrics` endpoint.

---

## Node.js Application Setup
- Below is a simple `Node.js` application that uses the `Express` framework along with the `swagger-stats` middleware to expose Prometheus metrics. 
- The application listens on port `3000` and defines several endpoints that return dummy data.
```js
const express = require("express");
const swStats = require("swagger-stats");
const app = express();

// Set up swagger-stats middleware to expose Prometheus metrics at /swagger-stats/metrics
app.use(swStats.getMiddleware());

app.get("/", (req, res) => {
    res.send("Hello World!");
});

app.get("/comments", (req, res) => {
    res.send("Comments");
});

app.get("/threads", (req, res) => {
    res.send("Threads");
});

app.get("/replies", (req, res) => {
    res.send("Replies");
});

app.listen(3000, () => {
    console.log("Server listening on port 3000...");
});
```

---

## Containerizing the Application
- To deploy the application in a containerized environment, create a Docker image using the following Dockerfile:
```Dockerfile
FROM node:16

# Create the application directory
WORKDIR /usr/src/app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install
# For production builds, consider using:
# Copy the remaining application code
COPY . .

# Expose the port on which the application listens
EXPOSE 3000

# Start the application
CMD ["node", "index.js"]
```

- Build Docker image with a descriptive tag and Docker Hub username. For example:
```bash
sudo docker build -t sanjeevkt720/prometheus-demo .
```

- Once built, push the image to Docker Hub:
```bash
$ sudo docker push sanjeevkt720/prometheus-demo

Using default tag: latest
The push refers to repository [docker.io/sanjeevkt720/prometheus-demo]
91f31d757fd7: Pushing [==================================================>]  92.67kB
be6b84f51e: Preparing
20de42680ea1: Preparing
58959534fd62: Preparing
65570773dc057: Preparing
3639ead5fca4: Waiting
595ddb2d3de6: Waiting
371d43a25876: Waiting
381f4a00ea68: Waiting
155c77c25cb5: Waiting
4d19f3efa378: Waiting
6ddff9eed369: Waiting
```

---

## Deploying on Kubernetes
- The next step is to deploy the containerized application on a Kubernetes cluster. 
- This involves creating Kubernetes manifests for both the Deployment and the Service.

### Deployment Configuration
- Create a manifest file (e.g., `api-deploy.yaml`) to define a Deployment that runs two replicas of your application. 
- The container image is pulled from Docker Hub, and port `3000` is exposed to match the port used by the application.
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  labels:
    app: api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: sanjeevkt720/prometheus-demo
          ports:
            - containerPort: 3000
```

### Service Configuration
- In the same manifest file, define a `ClusterIP` Service that enables pod-to-pod communication inside the cluster. 
- The Service will use the same label selector as the Deployment and will forward traffic on port 3000.
```yml
apiVersion: v1
kind: Service
metadata:
  name: api-service
  labels:
    job: node-api
    app: api
spec:
  type: ClusterIP
  selector:
    app: api
  ports:
    - name: web
      protocol: TCP
      port: 3000
      targetPort: 3000
```

- Apply the manifest file to the Kubernetes cluster with:
```bash
$ kubectl apply -f api-deploy.yaml

deployment.apps/api-deployment created
service/api-service created
```

### Verifying the Deployment
- After applying your manifests, verify that the Service and Deployment are running correctly.
```bash
$ kubectl get service
NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP  PORT(S)             AGE
api-service                   ClusterIP   10.100.40.38    <none>       3000/TCP           11s

$ kubectl get deployment
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
api-deployment   1/2     2            1           20s
```
