#### Pods

```cmd
# Create a container from nginx image
kubectl run <POD_NAME> --image=nginx

# Create a container from yaml file
kubectl create -f <FILE_NAME>.yaml

# Delete a container from yaml file
kubectl delete -f <FILE_NAME>.yaml
```

#### Deployment

```cmd
# Create a deployment
kubectl create deployment <DEPLOYMENT_NAME> --image=httpd

# Create a deployment with verbosity
kubectl create deployment <DEPLOYMENT_NAME> --image=httpd --v=10

# Look all the deployments, replicaSets and pods created
kubectl get deployment, rs, pods

# Get IP information on a pod
kubectl get pod -o wide

# Delete a deployment
kubectl delete deploy <DEPLOYMENT_NAME>
```

##### Lab 2.1. Overview
```cmd
# Download the tar.xz
wget https://cm.lf.training/LFD259/LFD259_V2025-09-22_SOLUTIONS.tar.xz --user=LFtraining --password=Penguin2014

# Update packages
sudo apt update && sudo apt install xz-utils -y

# Extract the tar.xz
tar -xvf LFD259_V2025-09-22_SOLUTIONS.tar.xz
```

##### Lab 2.2. Deploy a New Cluster
```cmd
# Find the location of a file
find $HOME -name k8scp.sh

# Console logs the file content
more LFD259/SOLUTIONS/s_02/k8scp.sh

# Copy the file to the current location
cp LFD259/SOLUTIONS/s_02/k8scp.sh .

# Execute bash script to generate control plane and nodes
bash k8scp.sh | tee $HOME/cp.out

# Need to do this in a complete new worker node 
find $HOME -name k8sWorker.sh
cp LFD259/SOLUTIONS/s_02/k8sWorker.sh .
bash k8sWorker.sh | tee worker.out

# Install a text editor
sudo apt-get install bash-completion -y

# Configure completion in the current shell
source <(kubectl completion bash)

# Ensure future shells have completion
echo "source <(kubectl completion bash)" >> $HOME/.bashrc

# Use kubectl help command
kubectl --help

# Look at the taints in nodes
kubectl describe nodes | grep -i taint

# Remove the taints (notice the - symbol in the end)
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

##### Lab 2.3. Create a Basic Pod
```cmd
# Create a Pod from a yaml file
kubectl apply -f <FILE_NAME>

# View the details of the created pod
kubectl describe pod <POD_NAME>

# Delete the running pod
kubectl delete pod <POD_NAME>

# Check internal IP assigned using -o wide option
kubectl get pods -o wide

# Create service
kubectl apply -f basicservice.yaml
```

##### Lab 2.4. Multi Container Pods
```cmd
# Delete a pod
kubectl delete pod <POD_NAME>

# Create a pod
kubectl apply -f <POD_FILE>
```

##### Lab 2.5. Create a Simple Deployment
```cmd
# Create a Deployment
kubectl create deployment <DEPLOYMENT_NAME> --image=nginx

# Check deployments and pods created in the default namespace
kubectl get deploy,pod\

# Describe a deployment
kubectl describe deploy <DEPLOYMENT_NAME>

# Get all the namespaces
kubectl get namespaces

# Get all the pods in a given namespace
kubectl get pods -n kube-system

# Get pods in all the namespaces
kubectl get pods --all-namespaces

# Delete the replicaset, but it will start again
kubectl delete rs <REPLICASET_NAME>

# Deleting a deployment will remove all the replicasets and the pods
kubectl delete deploy <DEPLOYMENT_NAME>
```

##### Lab 3.1. Deploy a New Application
```cmd
# Install python3
sudo apt-get -y install python3

# Locate python binary
which python3

# install podman
sudo apt-get install -y podman

# Create an image
sudo podman build -t simpleapp .
docker build -t simpleapp .

# Show all the images
sudo podman images
docker images

# Run a container
sudo podman run localhost/simpleapp
docker run simpleapp:latest
```

##### Lab 3.2. Configure a Local Repo
```cmd
# Spin up containers 
kubectl create -f easyregistry.yaml

# Create a deployment
kubectl create deployment <DEP_NAME> --image=nginx

# Scale a deployment
kubectl scale deployment <DEP_NAME> --replicas=6
```

##### Lab 3.3. Configure Probes
```cmd
# Convert the deployment to yaml format
kubectl get deployment <DEP_NAME> -o yaml > simpleapp.yaml

# Create a deployment from given file
kubectl create -f simpleapp.yaml

# Execute into a pod
kubectl exec -it <POD_NAME> -- /bin/bash

# Add the /tmp/healthy file to all containers
for name in $(kubectl get pods -l app=simpleapp -o name); do
  kubectl exec $name -- sh -c 'touch /tmp/healthy'
done

# Tail a pod
kubectl describe pod <POD_NAME> | tail
```