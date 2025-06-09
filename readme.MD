## Steps to deploy the application in minikube 
## you need to install minikube for linux
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```
## install kubectl
```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
# Validate the binary (optional)

### Download the kubectl checksum file:
```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```
### Validate the kubectl binary against the checksum file:
```bash
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```
#### the output is ok

## Install kubectl
```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

```
## Test to ensure the version you installed is up-to-date:
```bash
kubectl version --client
```
## start the minikube cluster 
```bash
minikube start
```
## see the version 
```bash 
minikube version
```
## minikube status 
```
minikube status
```
### pulling the image from  Harbor private repository we need to configure the files
#### enter into minikube sheel
```bash
minikube ssh

```
## do this configurations
```bash
cat /etc/docker/daemon.json
```
## replace the file with below data
```bash
{
  "dns": ["8.8.8.8"],
  "insecure-registries": ["ip-addess:port"]
}

```
#### ip-addess:port = here we can add private Docker Registry(ex harbor,docker
#
```bash
cat /lib/systemd/system/docker.service
```
##### replace the file with below data
```bash
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
BindsTo=containerd.service
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket
StartLimitBurst=5
StartLimitIntervalSec=120
[Service]
Type=notify
Restart=on-failure
# Clear out inherited ExecStart configuration and set the correct parameters
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2376 -H unix:///var/run/docker.sock --default-ulimit=nofile=1048576:1048576 --debug --label provider=docker
# Optional: remove insecure registry if not needed
# --insecure-registry 10.96.0.0/12 (optional, remove if not needed)
# Resource limits for Docker to avoid OOM issues
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Time to wait before timing out the start
TimeoutStartSec=90
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
Delegate=yes
KillMode=process
[Install]
WantedBy=multi-user.target

```
##### after this restart deamon and docker 
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
### after this exit from shell 
```bash 
exit
```
## write the deployment.yaml file to deploy the frontend application in minikube
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 1  # Number of Pods to run
  selector:
    matchLabels:
      app: frontend  # Updated label to match Pods
  template:
    metadata:
      labels:
        app: frontend  # Label updated to match Pods
    spec:
      containers:
        - name: frontend-container  # Container name updated
          image: ip-address:port/project/frontend # Updated image path
          ports:
            - containerPort: 3000  # Port your app will run on inside the container
      imagePullSecrets:
        - name: my-harbor-secret  # Secret for accessing Harbor (private registry)

---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service  # Service name
spec:
  selector:
    app: frontend  # Selector to match the app label in Deployment
  ports:
    - port: 3000        # External port exposed on the service (host)
      targetPort: 3000   # Port inside the container where the app runs (container)
      nodePort: 3000     # Expose the service on the node port 3000
  type: LoadBalancer  # Expose the service externally, useful for cloud environments
```
## create secret file to login harbor
```bash
kubectl create secret docker-registry my-harbor-secret \
  --docker-server=<private-registry-url> \
  --docker-username=<your-username> \
  --docker-password=<your-password> 

```
### after all these steps
#### deploy the application in minikube 
```bash 
minikube apply -f deployment.yaml

```
#### see the deployment
```bash 
kubectl get deployment
```
#### get the pods
```bash
kubectl get pods
```
#### get services
```bash
kubectl get service
```
## Here we are using service as load balancer
## To acess the deployed application we can do port forward
## In current terminal enter
```bash
minikube tunnel
```
### open the another terminal forward the port uding following command
```bash
 kubectl port-forward --address 0.0.0.0 service/ivlfront-deployment 8082:3000
```
## open the browser and access your application using localhost:8082




