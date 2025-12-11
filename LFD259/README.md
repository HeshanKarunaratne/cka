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

##### Lab 2.2. Deploy a New Cluster
```cmd
# Download the tar.xz
wget https://cm.lf.training/LFD259/LFD259_V2025-09-22_SOLUTIONS.tar.xz --user=LFtraining --password=Penguin2014

# Update packages
sudo apt update && sudo apt install xz-utils -y

# Extract the tar.xz
tar -xvf LFD259_V2025-09-22_SOLUTIONS.tar.xz
```
