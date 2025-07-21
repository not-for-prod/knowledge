# Gitlab integration with k8s cluster

https://www.youtube.com/watch?v=fwtxi_BRmt0

## Configure agent, [guide](https://docs.gitlab.com/user/clusters/agent/install/):

1. Create `k8s-connection` project in your gitlab
2. In the repository, in the default branch, create an agent configuration file at:

    ```
    .gitlab/agents/<agent-name>/config.yaml
    ```

    ex. 

     ```
    .gitlab/agents/k8s-connection/config.yaml
    ```
   
3. [Register the agent with GitLab](https://docs.gitlab.com/user/clusters/agent/install/#option-1-agent-connects-to-gitlab)
4. Connect a Kubernetes cluster, choose created on `step 1`
5. [Install the agent into the cluster](https://docs.gitlab.com/user/clusters/agent/install/#install-the-agent-in-the-cluster)

## Fill registry with images

### Create `Dockerfile`. Example for Golang

```Dockerfile
# Step 1: Build the Go app
FROM golang:1.24.1-alpine3.21 AS builder

# Set the Current Working Directory inside the container
WORKDIR /app

# Copy the Go Modules file
COPY go.mod go.sum ./

# Download all dependencies. Dependencies will be cached if the go.mod and go.sum are not changed
RUN go mod tidy

# Copy the entire local directory to the working directory inside the container
COPY . .

# Build the Go app
RUN go build -o main ./cmd/main.go

# Step 2: Create the final image
FROM alpine:latest

WORKDIR /root/

# Copy the Pre-built binary file from the builder stage
COPY --from=builder /app/main .

# Expose port 8080
EXPOSE 8000
EXPOSE 9000

## Command to run the executable
ENTRYPOINT ["./main"]
```

### Push image into container registry, [guide](https://docs.gitlab.com/user/packages/container_registry/build_and_push_images/) 

Go to

`SideBar > Deploy > Container Registry`

Here (in right corner) you can see commands to push image

```shell
docker login registry.gitlab.com
docker build -t <your gitlab project name> .
docker push registry.gitlab.com/<your gitlab project path>
```

Create `.gitlab-ci.yml` in your project with build stage:

> fill `<your gitlab project path>` with one from the previous step

```yaml
stages:
  - build

build_image:
  image: docker:20.10.16
  stage: build
  services:
    - docker:20.10.16-dind
  script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login $CI_REGISTRY -u $CI_REGISTRY_USER --password-stdin
    - docker build -t $CI_REGISTRY/<your gitlab project path>:latest .
    - docker push $CI_REGISTRY/<your gitlab project path>:latest
```

## Create k8s infrastructure

1. Create `docker-registry` secret

```shell
kubectl create secret docker-registry <secret-name> --docker-server=registry.gitlab.com --docker-username=<username> --docker-password='<password>' --dry-run=client -o yaml > secret.yaml
kubectl apply -f secret.yaml
```
   
2. Create `configmap` 

```shell
kubectl create configmap <your gitlab project name>-configmap --from-file=./config.yaml
```
   
3. Create `deployment`

```shell
kubectl create deployment <deployment-name> --image=registry.gitlab.com/<your gitlab project path>:latest --dry-run=client -o yaml > deployment.yaml
```

modify `deployment.yaml` with secret and config volume from steps 1 and 2. Your `deployment.yaml` should look like:

```yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <name>-deployment
  namespace: default
  labels:
    app: <deployment-name>
spec:
  replicas: 1
  selector:
    matchLabels:
      app: <deployment-name>
  template:
    metadata:
      labels:
        app: <deployment-name>
    spec:
      containers:
        - name: <deployment-name>-container
          image: registry.gitlab.com/<your gitlab project path>:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - name: config-volume
              mountPath: /root/config
              readOnly: true
      volumes:
        - name: config-volume
          configMap:
            name: <deployment-name>-configmap
      imagePullSecrets:
        - name: <secret-name> 
```

4. Create `service`

```shell
kubectl create service clusterip <service-name> --tcp='8000:8000' --dry-run=client -o yaml > service.yaml
```

modify `service.yaml` adding more tcp ports (if you need)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: <service-name>
  namespace: default
  labels:
    app: <service-name>
spec:
  type: ClusterIP
  selector:
    app: <service-name>
  ports:
    - name: http-private
      protocol: TCP
      port: 8000
      targetPort: 8000
    - name: http-public
      protocol: TCP
      port: 9000
      targetPort: 9000
```

when all deployment yamls are ready, add them into `.deploy` folder and modify `.gitlab-ci.yml` adding deploy stage.
Your final `.gitlab-ci.yml` would look like

```yaml
stages:
  - build
  - deploy

build_image:
  image: docker:20.10.16
  stage: build
  services:
    - docker:20.10.16-dind
  script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login $CI_REGISTRY -u $CI_REGISTRY_USER --password-stdin
    - docker build -t $CI_REGISTRY/<your gitlab project path>:latest .
    - docker push $CI_REGISTRY/<your gitlab project path>:latest

deploy_project:
  image:
    name: bitnami/kubectl:latest
    entrypoint: ['']
  stage: deploy
  script:
    - kubectl config use-context <agent-path>/k8s-connection:k8s-connection
    - kubectl get pods
    - kubectl get nodes -o wide
    - kubectl apply -f $CI_PROJECT_DIR/.deploy
```

if you need to handle incoming HTTP/HTTPS requests you need `ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: ingress1.example.com
      http:
        paths:
          - path: /svc
            pathType: Prefix
            backend:
              service:
                name: <service-name>
                port:
                  number: 80
```

and `loadbalancer.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
spec:
  selector:
    app.kubernetes.io/name: ingress-nginx
  ports:
    - name: http
      port: 80
      targetPort: 80
    - name: https
      port: 443
      targetPort: 443
  type: LoadBalancer
```