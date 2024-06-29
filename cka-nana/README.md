# CKA - Nana

### Node
- Node is a physical or a virtual machine

#### Master Node
- All the managing processes are done by the Master Nodes(Control Plane)
    - Scheduling pods
    - Monitoring
    - Re-scheduling/Re-starting pods
    - Join new node
- There are 4 processes that must be installed on every master node
    - Api Server: Cluster gateway, acts as a gatekeeper for authentication
    - Scheduler: Decides on which node new pod should be scheduled
    - Controller Manager: Detects cluster state changes
    - etcd: Key value store
- Master processes are much more important and there is only a handful of master processes, so they need less CPU, RAM and STORAGE

#### Worker Node
- There are 3 processes that must be installed on every worker node
    - Container runtime: docker, containerd, cri-o
    - kubelet: Interacts with the both the container and the node, kubelet starts the pod with a container inside
    - Kube proxy
- Worker nodes do the actual job of running those workload, so they require more resources

#### Pod
- Smallest unit in kubernetes is a pod and usually 1 application per pod is recommended
- Each pod gets an IP address, not the container
- Pods can communicate with each other using the IP address
- Pods are ephemeral, everytime a pod die a new pod will be created with a `new IP address`

#### Service
- Is a permanent IP address
- Lifecycle of Pod and Service is `not connected`

#### ConfigMap
- Externalize configurations of the application
- ConfigMap is for non confidential data only

#### Secrets
- Use Secrets for confidential data

#### Volumes
- Storage on local machine, or remote(outside of the K8s cluster)

#### Deployment
- Blueprint for pods 
- For `Stateless` Apps
- DBs cant be replicated via Deployments

#### StateFulSet
- DBs, ElasticSearch or any other medium should be deployed as a `StateFulSet`

#### DaemonSet
- Calculates how many replicas are needed based on existing nodes
- Deploys just 1 replica per Node
- No need to define replica count, automatically scales up and down

#### Chmod
- 0 - No permission
- 1 - Read
- 2 - Execute
- 3 - Read and execute
- 4 - Write
- 5 - Write and Read
- 6 - Write and execute
- 7 - Read, Write and execute

### Encryption

#### Symmetric Key
- Same key used to encrypt and decrypt
- Faster compared to Asymmetric encryption
- 2 major problems are,
    - Key needs to be stored securely
    - Secure channel is required to transfer the key

#### Asymmetric Key
- Uses a public key and a private key
- Slower compared to symmetric encryption

#### Creating Kubernetes cluster
1. Create 3 nodes(master-node, worker-node-1, worker-node-2)
    - Make sure worker nodes have more resources than master node
2. Deploy container runtime, kube proxy and kubelet on each node
3. Deploy api server, scheduler, controller  manager and etcd pods in master node

#### Namespace
- Organize resources in namespaces
- Virtual cluster inside a cluster
- Use cases
    1. Structure components
    2. Avoid conflicts between teams
    3. Share services between different environments
    4. Resource limits on namespace level
```cmd
kubectl create namespace $name_space
kubectl get namespaces
kubectl get pod -n $namespace
kubectl describe svc $service_name
kubectl get ep
kubectl get deployment --show-labels

- Get all the services with the label
kubectl get svc -l app=nginx

kubctl scale --replicas=2 -f nginx-service.yml

- Record the history
kubectl scale deployment --replicas=5 nginx-deployment --record

- Check the history
kubectl rollout history deployment/nginx-deployment

- Open up bash shell for a pod
kubectl exec -it test-nginx-service -- bash

- Check if pod to pod communication works
curl $cluster_ip:$cluster_port

- Check all the default values when kubeadm execute
kubeadm config print init defaults

- Create a new service without yml file
kubectl create service clusterip $service_name --tcp=80:80

- Create an alias which is only applicable for that session
alias k=kubectl

- Creating a sample service, not executing it and saving it as a yaml file
kubectl create service clusterip $service_name --tcp=80:80 --dry-run=client -o yaml > myService.yml

- Create a sample ingress
kubectl create ingress my-app-ingress --rule=host/path=service:port --dry-run=client -o yaml > my-ingress.yml

- Describe cluster role
kubectl describe clusterrole $cluster_role_name

- Create a clusterrolebinding to map clusterrole per specific user/group
kubectl create clusterrolebinding dev-crb --clusterrole=dev-cr --user=tom --dry-run=client -o yaml > dev-crb.yml

- Using auth sub commands
kubectl auth can-i get node
kubectl auth --kubeconfig config can-i get node --as $user_name

- Create a service account
kubectl create serviceaccount jenkins --dry-run=client -o yaml > jenkins-sa.yml

- Create a cicd role
kubectl create role cicd-role --verb=create,list,update --resource=deployments.apps,services --dry-run=client -o yaml > cicd-role.yml

- Checking logs in a sidecar container
kubectl logs nginx-deployment-64c549f7f5-ndw2f -c log-sidecar

- Restart all the pods in a deployment/statefulSet
kubectl rollout restart deployment/my-db

- Labeling different resources
kubectl label node $node_name type=cpu

- Can check the node for taints
kubectl describe node master
```

- Below yaml format is a better way to create namespace with resources
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-configmap
  namespace: my-namespace
```

#### Networks
- Every pod gets its own unique IP address
- Pods on same node can communicate with that IP address
- Pods on different node can communicate with that IP address without NAT(Network Address Translation)

#### Labels
- Use cases of using labels
    1. Label to identify and target any kubernetes component

#### Fully Qualified Domain Name
- When you try to access a pod from a different namespace you need to use the FQDN like below
    - $service_name.$namespace.svc.cluster.local <= FQDN

#### Ingress
- Usages
    1. Configure secure connection
    2. Loadbalancing to different services
    3. Configure routing
    4. Single entrypoint for all the resources

### RBAC(Role Based Access Control)

#### Role
- With Role component you can define namespaced permissions
- Bound to a specific Namespace
- What resources in that namespace you can access?
    - Pod/Deployment/Service/etc
- What action you can do with this resources?
    - list/get/update/delete/etc
- `No information on WHO gets these permissions` - You need to use Role Binding for that(Bind to user of group)

#### ClusterRole
- Defines resources and permissions cluster wide
- Use `ClusterRoleBinding` component to bind these roles to cluster groups and users

#### SideCar Container
- You can have multiple containers inside a pod, where the container providing helper functionality is called the sidecar container
- They are communicate without needing a service name

#### Persistent Volumes
- PV are resources that need to be there before the pod that depends on it is created

#### Persistent Volume Claim
- Application has to claim the PV
- Pod requests the volume through the PV claim

#### ConfigMaps and Secrets
- Pods can consume ConfigMaps or Secrets in 2 different ways
    - As individual values using Env variables
    - As configuration files using Volumes

#### nodeName
- Tells the scheduler to schedule the pod on a given node

#### Taints
- Nodes that have Taints stops pods from scheduling inside that nodes if they have specific keyValues defined
- Master node have Taints to stop pods scheduling inside node
- Kubeadm sets the taints when bootstrapping

#### Tolerations
- Applied to pods
- Allow the pods to schedule onto nodes with matching Taints

#### Inter-Pod Anti-Affinity
- Pod should not run on a node if that node is already runnning one or more pods with a specific label

#### Inter-Pod Affinity
- Pod should run on a node if that node is already runnning one or more pods with a specific label

#### Liveness-Probe
- Health checks only after the container started
- 3 ways to check the health status
    1. Executing probe - execute specific command
    2. TCP probe - connect to tcp socket
    3. HTTP probe - sends a request to specified port and path

#### Readiness-Probe
- It tells kubernetes that it is ready to listen to traffic

#### Context
```cmd
- Display list of context
kubectl config get-contexts

- Display the current context
kubectl config current-context

- Set context
kubectl config set-context --current --namespace=kube-system
```

#### Network Policies
- Which pods can talk to which pods

```cmd
kubectl create ns myapp
kubectl apply -f .
kubectl config set-context --current --namespace myapp

```