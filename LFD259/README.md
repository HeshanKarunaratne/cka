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
