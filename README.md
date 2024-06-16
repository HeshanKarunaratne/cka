# CKA

#### Nodes(Minions)
Containers will be launched in a node. What if the node on which our applications are running fails. Our application goes down as well. So you need to have more than 1 node.

#### Cluster
A cluster is a set of nodes grouped together. This way even if one node fails you have your application accessible from the other nodes. Helps to share load as well. Master watches over the nodes in the cluster and is responsible for the actual orchestration of containers on the worker nodes.

#### Master vs Worker Nodes

##### Master
Contains
- kube-apiserver: Acts as the frontend for kubernetes, talks to the api server to interact with the cluster
- etcd: Distributed key-value store to store all data used to manage the cluster
- controller: Noticing and responding when nodes, containers or endpoints goes down
- scheduler: Distributing work or containers across multiple nodes

##### Worker
Contains
- kubelet: Is the agent responsible for making sure that the containers are running on the node as expected
- container runtime: Underline software used to run containers

#### Pods
Kubernetes does not deploy containers directly on the worker nodes, the containers are encapsulated into a kubernetes object known as a Pod. The containers inside a pod will have access to the same storage, the same network namespace and same fate(created together, destroyed together)

#### Kubectl commands
```cmd
- Run an applicatio
kubectl run hello-minikube

- Get Cluster information
kubectl cluster-info

- List all the nodes part of the cluster
kubectl get nodes
```