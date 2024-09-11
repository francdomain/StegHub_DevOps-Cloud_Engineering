# Self Side Task

1. Build the Tooling app Dockerfile and push it to Dockerhub registry

2. Write a Pod and a Service manifests, ensure that you can access the Tooling app's frontend using port-forwarding feature.


## Step 1: Build the Tooling App Dockerfile and Push it to DockerHub

1. Build the Docker Image

![](./images/d-compos-build.png)

||||||||||||||||















Create a pod manifest file for the tooling app - `tooling-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tooling-pod
  labels:
    app: tooling-app
spec:
  containers:
  - name: tooling-container
    image: francdocmain/tooling-app:latest
    ports:
    - containerPort: 80
      protocol: TCP
```
![]()

Apply the manifest with kubectl

```bash
kubectl apply -f tooling-pod.yaml
```

Create a service manifest file for the tooling app - `tooling-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tooling-service
spec:
  selector:
    app: tooling-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
![]()

Apply the manifest with kubectl

```bash
kubectl apply -f tooling-service.yaml
```





