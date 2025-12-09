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