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