#### Pods

```cmd
# Create a container from nginx image
kubectl run <POD_NAME> --image=nginx

# Create a container from yaml file
kubectl create -f <FILE_NAME>.yaml

# Delete a container from yaml file
kubectl delete -f <FILE_NAME>.yaml
```