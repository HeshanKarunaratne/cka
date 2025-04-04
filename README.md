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
- controller: Notices and responds when nodes, containers or endpoints goes down
- scheduler: Distributing work or containers across multiple nodes, looks for newly created containers and assigns them to nodes

##### Worker
Contains
- kubelet: Is the agent responsible for making sure that the containers are running on the node as expected
- container runtime: Underline software used to run containers

```cmd
kubectl run hello-minikube
kubectl cluster-info
kubectl get nodes
```

#### Pods
Kubernetes does not deploy containers directly on the worker nodes, the containers are encapsulated into a kubernetes object known as a Pod. The containers inside a pod will have access to the same storage, the same network namespace and same fate(created together, destroyed together)

- A pod have a one to one relationship with the containers
- A pod should have mandatory apiVersion, kind, metadata and spec 
- Metadata is a dictionary and it should have name and labels under it(what kubernetes expect)
- Labels is also a dictionary and you can have many key value pairs for it
- Spec is a dictionary and you can place multiple containers inside the container(List/Array element) tag

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
```

```cmd
kubectl apply -f pod-definition.yml
kubectl get pods
kubectl describe pod myapp-pod
```

##### Questions - Pods
```cmd
- Check running pods
kubectl get pods

- Create a new pod with the nginx image
kubectl run nginx --image=nginx

- How many pods in default namespace?
kubectl get pods --namespace default

- What is the image used to create the new pods?
kubectl describe pod newpods-92tl5

- Which nodes are these pods placed on?
kubectl get pods -o wide

- What images are used in the new webapp pod?
kubectl describe pod webapp
And get how many containers are present

- What does the READY column in the output of the kubectl get pods command indicate?
Running container count/ Total container count

- Delete the webapp Pod
kubectl delete pod webapp

- Create a new pod with the name redis and the image redis123?
kubectl run redis --image=redis123 --dry-run=client -o=yaml > redis.yaml
kubectl apply -f redis.yaml

- Create a pod definition file from a pod?
kubectl get pod <pod-name> -o yaml > pod-definition.yaml
```

#### Replication Controller
Ensures that the specified number of pods are running at all times. To create multiple pods to share the load across them. When the number of users increase we deploy additional pods to balance the load across the pods. If the demand further increases and if we were to run out of resources on the first node we could deploy additional pods across the other nodes in the cluster.

```yml
apiVersion: v1
kind: ReplicationController
metadata: 
  name: myapp-rc
  labels: 
    app: myapp
    type: front-end
spec: 
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
```

```cmd
kubectl apply -f rc-definition.yml
kubectl get replicationcontroller
kubectl get pods
kubectl delete replicationcontroller myapp-rc
```

#### ReplicaSet
- It is mandatory to have a selector tag under spec if using ReplicaSet. This is the difference between ReplicationController and ReplicaSet. If you have created pods before creating the ReplicaSet, you can use the selector to identify those pods, or else you can add the pod-definition inside the template
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: myapp-replicatset
  labels: 
    app: myapp
    type: front-end
spec: 
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

```cmd
kubectl apply -f replicaset-definition.yml
kubectl get replicaset
kubectl get pods
kubectl delete replicaset myapp-replicatset
kubectl describe replicaset
kubectl delete pod $pod_name

- How to update the defintion.yaml file?
kubectl replace -f definition.yaml
kubectl scale --replicas=6 -f definition.yaml
kubectl scale --replicas=6 <TYPE> <NAME>
```

##### Questions - ReplicaSet
```cmd
- How many ReplicaSets exist on the system?
kubectl get replicasets

- What is the image used to create the pods in the new-replica-set?
kubectl describe replicaset new-replica-set

- Delete any one of the 4 PODs?
kubectl delete pod new-replica-set-55nxs

- Delete the two newly created ReplicaSets - replicaset-1 and replicaset-2?
kubectl delete rs replicaset-1 replicaset-2

- Scale the ReplicaSet to 5 PODs?
kubectl scale --replicas=5 rs new-replica-set

- Now scale the ReplicaSet down to 2 PODs?
kubectl edit rs new-replica-set 
```

#### Labels and Selectors
We can use the labels as a filter to selector. This way replicaset will know which pods to monitor.
```yml
metadata: 
  name: myapp-pod
  labels: 
    tier: front-end
```

```yml
selector:
  matchLabels:
    tier: front-end
```

#### Scaling
```cmd
kubectl replace -f replicatset-definition.yml
kubectl scale --replicas=6 -f replicatset-definition.yml
```

#### Deployments

```yml
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: myapp-deployment
  labels: 
    app: myapp
    type: front-end
spec: 
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

```cmd
kubectl create -f deployment-definition.yml
kubectl get deployments
kubectl get replicasets
kubectl get pods
kubectl describe deployment $deployment_name
kubectl get all
```

##### Questions - Deployment
```cmd
- How many deployments exists in the system?
kubectl get deployments

- What is the image used to create the pods in the new deployment?
kubectl describe deployment frontend-deployment

- Create a new Deployment with the below attributes using your own deployment definition file. Name: httpd-frontend; Replicas: 3; Image: httpd:2.4-alpine
kubectl create deployment httpd-frontend --replicas=3 --image=httpd:2.4-alpine
```

#### Namespaces
- Kubernetes creates default, kube-system and kube-public namespaces during startup, so that we wont accidently delete any resources
```text
db-service   . dev       . svc     . cluster.local
Service Name   Namespace   Service   Domain   
```

- Create a new namespace
```yaml
apiVersion: v1
kind: Namespace
metadata: 
  name: dev
```

```cmd
- Get pods in a different namespace
kubectl get pods --namespace=kube-system

- Create a namespace
kubectl create namespace <NAMESPACE_NAME>

- Set the namespace to 'dev' so that you dont need to specify this in each call
kubectl config set-context $(kubectl config current-context) --namespace=dev

- Get pods in all the namespaces
kubectl get pods --all-namespaces
```

- Create resource quota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata: 
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

##### Questions - Namespaces
```cmd
- How many Namespaces exist on the system?
kubectl get ns

- How many pods exist in the research namespace?
kubectl get pods --namespace=research

- Create a POD in the finance namespace?
kubectl run redis --image=redis --namespace=finance

- Which namespace has the blue pod in it?
kubectl get pods --all-namespaces
kubectl get pods -A

- What DNS name should the Blue application use to access the database db-service in its own namespace?
Use the service name

- What DNS name should the Blue application use to access the database db-service in the dev namespace?
db-service.dev.svc.cluster.local
```

```yml
apiVersion: v1
kind: Pod
metadata: 
  name: myapp-pod
  namespace: dev
  labels: 
    app: myapp
    type: front-end
spec: 
  containers:
  - name: nginx-container
    image: nginx
```

#### Imperative Commands
```cmd
- Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
kubectl run nginx --image=nginx --dry-run=client -o yaml

- Create a deployment with 4 replicas
kubectl create deployment --image=nginx nginx --replicas=4

- Scale a deployment
kubectl scale deployment nginx --replicas=4

- Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml

- Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
```

##### Questions - Imperative Commands
```cmd
- Deploy a pod named nginx-pod using the nginx:alpine image?
kubectl run nginx-pod --image=nginx:alpine

- Deploy a redis pod using the redis:alpine image with the labels set to tier=db?
kubectl run redis --image=redis:alpine --labels tier=db

- Create a service redis-service to expose the redis application within the cluster on port 6379?
kubectl create service clusterip --tcp=6379:6379 redis-service

- Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas?
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3

- Create a new pod called custom-nginx using the nginx image and run it on container port 8080?
kubectl run custom-nginx --image=nginx --port=8080

- Create a new namespace called dev-ns?
kubectl create ns dev-ns

- Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas?
kubectl create deployment redis-deploy --namespace=dev-ns --image=redis --replicas=2

- Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80?
kubectl run httpd --image=httpd:alpine --port=80 --expose=true
```

#### Updates and Rollbacks
When you first create a deployment it triggers a rollout. A new rollout creates a new deployment revision. In the future when the application is upgraded, a new rollout is triggered and a new deployment revision is created.

```cmd
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl apply -f deployment-definition.yml
kubectl setimage deployment/myapp-deployment nginx=nginx:1.9.1
kubectl describe deployment myapp-deployment
kubectl rollout undo deployment/myapp-deployment
kubectl run nginx --image=nginx
kubectl get deployments
kubectl create -f deployment-definition.yml --record
```

- Deployment Strategies
  1. Recreate: Destroy all the old versions and create the new versions. There will be an application downtime
  2. Rolling update(default): Incremently bring down old version/create new version at a time. In this way application never goes down


#### Networks
IP address is assigned to a pod. All nodes can communicate with all containers and vice versa without using a NAT. Internal pod network is in the range of 10.244.0.0

#### Services
Kubernetes services enable communication between various components within and outside of the application.

#### NodePort
Service makes an internal pod accessible on a port on the pod

<img src="images/nodeport_service.PNG" alt="Alt text" width="800" height="400">

There are 3 ports involved
  - Target Port(80): The port on the pod where the actual web server is runnning
  - Port(80): The port on the service
  - NodePort(30008): The node port(30000-32767)

```yaml
apiVersion: v1
kind: Service
metadata: 
  name: myapp-nodeport-service
spec: 
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```
- 'port' is the only mandatory field and ports is an array

```cmd
kubectl create -f nodeport-service-definition.yml
kubectl get services
```

In any case whether its a single pod in a single node, multiple pods in a single node, multiple pods in multiple nodes the service is created exactly same. When pods are removed or added the service is automatically updated.

- Get the ip using `ipconfig` and use `curl $ipconfg:30008`
#### ClusterIP
Service creates a virtual IP inside the cluster to enable communication between differnet services

```yml
apiVersion: v1
kind: Service
metadata: 
  name: myapp-clusterip-service
spec: 
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
```

Service can be accessed by other pods using cluster ip or the service name

#### LoadBalancer
Provisions a load balancer for our application

#### Microservices

```cmd
docker run -d --name=redis redis
docker run -d --name=db -e POSTGRES_PASSWORD=123 postgres:16
docker run -d --name=vote -p 5000:80 --link redis:redis saiachyuthm/voting-app
docker run -d --name=result -p 5001:80 --link db:db saiachyuthm/result-app
docker run -d --name=worker --link redis:redis ---link db:db cfjaramillo/worker-app
```

- Steps
  1. Creating the pods
    - voting-app-pod exposing containerPort 80
    - worker-app-pod not exposing any ports
    - result-app-pod exposing containerPort 80
    - redis-pod exposing containerPort 6379
    - postgres-pod exposing containerPort 5432

  2. Creating the services
    - Internal
      - redis-service exposing port 6379, targetPort 6379, selectors of redis-pod and name as `redis`
      - postgres-service exposing port 5432, targetPort 5432, selectors of postgres-pod and name as `db`
    - External
      - voting-app-service exposing port 80, targetPort 80, selectors of voting-app-pod and type as LoadBalancer
      - result-app-service exposing port 80, targetPort 80, selectors of result-app-pod and type as LoadBalancer

#### Kubectl commands
```cmd
- Run an applicatio
kubectl run hello-minikube

- Get Cluster information
kubectl cluster-info

- List all the nodes part of the cluster
kubectl get nodes

- Run a nginx pod
kubectl run nginx --image nginx

- Check created pod
kubectl describe pods nginx
```

#### Docker CMD and ENTRYPOINT
- If you use a command with CMD it is hard coded, so will get the same output again and again
- To pass variable from command line use ENTRYPOINT
- Use ENTRYPOINT to pass value and CMD to keep a default value

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

```cmd
docker run --name ubuntu-sleeper ubuntu-sleeper 10
```

```yml
apiVersion: v1
kind: Pod
metadata: 
  name: ubuntu-sleeper-pod
spec: 
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep2.0"]
    args: ["5"]
```

- DOCKER     | KUBERNETES
- ENTRYPOINT | command
- CMD        | args

#### Questions - Test Commands and Arguments
```text
- Inspect the two files under directory webapp-color-3. What command is run at container startup?

- Dockerfile
FROM python:3.6-alpine
RUN pip install flask
COPY . /opt/
EXPOSE 8080
WORKDIR /opt
ENTRYPOINT ["python", "app.py"]
CMD ["--color", "red"]

- webapp-pod.yml
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["--color","green"]

--color green

- Create a pod with the given specifications. By default it displays a blue background. Set the given command line arguments to change it to green
kubectl run webapp-green --image=kodekloud/webapp-color -- --color green

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: webapp-green
  name: webapp-green
spec:
  containers:
  - image: kodekloud/webapp-color
    name: webapp-green
    args: ["--color", "green"]
```

#### Environement variables

```yml
apiVersion: v1
kind: Pod
metadata: 
  name: ubuntu-sleeper-pod
spec: 
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    env:
    - name: APP_COLOR
      value: red
```

#### ConfigMaps

```cmd
kubectl create configmap $config_name --from-literal=APP_COLOR=blue
kubectl create configmap $config_name --from-file=<path-to-file>
kubectl get configmaps
kubectl describe configmaps
```

```yml
apiVersion: v1
kind: ConfigMap
metadata: 
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

- Whole configmap
```yml
apiVersion: v1
kind: Pod
metadata: 
  name: ubuntu-sleeper-pod
spec: 
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    envFrom:
    - configMapRef:
      name: app-config
```
- Single key value
```yml
apiVersion: v1
kind: Pod
metadata: 
  name: ubuntu-sleeper-pod
spec: 
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_COLOR
```

##### Questions - ConfigMaps
```cmd
- What is the environment variable name set on the container in the pod?
kubectl describe pod webapp-color

- How many ConfigMaps exists in the default namespace?
kubectl get configmaps

- Identify the database host from the config map db-config?
kubectl describe configmap db-config

- Create a configmap?
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard
```

#### Secrets
- Store sensitive information
- Secrets are not encrypted , only encoded
- Configure RBAC for secrets, anyone who is able to create pods/deployments in the same namespace can access the secrets

```cmd
kubectl create secret generic app-secret --from-literal=DB_HOST=mysql
echo -n 'text' | base64
kubectl describe secrets
kubectl get secret app-secret -o yaml
echo -n 'gsgs=' | base64 --decode
```

```yml
apiVersion: v1
kind: Secret
metadata: 
  name: app-secret
data:
  APP_COLOR: blue
  APP_MODE: prod
```

##### Questions - Secrets
```cmd
- How many Secrets exist on the system?
kubectl get secrets

- How many secrets are defined in the dashboard-token secret?
kubectl get secret dashboard-token

- The reason the application is failed is because we have not created the secrets yet. Create a new secret named db-secret with the data given below. Name: db-secret; DB_Host=sql01; DB_User=root; DB_Password=password123
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --
from-literal=DB_Password=password123
```

#### Encrypting secret data at REST
```cmd
- Create a secret
kubectl create secret generic my-secret --from-literal=key1=supersecret

- Check the created secret
kubectl get secret my-secret -o yaml

- Decode and see the secret(anyone have access to do this)
echo -n 'c3VwZXJzZWNyZXQ=' | base64 --decode

- Install etcdctl
apt-get install etcd-client

- Still you can see the secret value in plain text in etcd, thats why we need to encrypt it
ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key get /registry/secrets/default/my-secret | hexdump -C

```

#### Container Security
```cmd
docker run --user=1001 ubuntu sleep 3600
docker run --cap-add MAC_ADMIN ubuntu
docker run --cap-drop MAC_ADMIN ubuntu
```

- Pod level security
```yml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      commands: ["sleep", "3000"]
```

- Container level security: Capabilities are only applicable at container level
```yml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      commands: ["sleep", "3000"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```

##### Questions - Security Contexts
```cmd
- What is the user used to execute the sleep process within the ubuntu-sleeper pod?
kubectl edit pod ubuntu-sleeper
kubectl exec ubuntu-sleeper -- whoami

- How to replace a pod without deleting it
kubectl replace --force -f /tmp/kubectl-edit-3136779847.yaml
kubectl exec ubuntu-sleeper -- whoami
```

#### Service Accounts
- There are 2 types of accounts in kubernetes
  1. User Accounts - Used by humans
    eg: Admin performing administrative task, developer accessing the cluster to deploy application
  2. Service Accounts - Used by an application
    eg: Prometheus uses kubernetes service account to pull the kubernetes api for performance metrics, jenkins to deploy application to kubernetes cluster

- When a service account is created, it creates a token and you can use this token as a Bearer token when making API calls to kubernetes cluster
- Kubernetes mount default service account to pods automatically

```cmd
- Create a service account
kubectl create serviceaccount $service_account_name

- Retrieve service account
kubectl get serviceaccount

- Describe a service account
kubectl describe serviceaccount $service_account_name

- Use the token for api calls
curl https://192.168.56.70:6555/api -insecure --header "Authorization: Bearer $token"
```

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-dashboard
      image: my-dashboard
  serviceAccountName: dashboard-sa
```

##### Questions - Service Accounts
```cmd
- How many Service Accounts exist in the default namespace?
kubectl get serviceaccount

- We just deployed the Dashboard application. Inspect the deployment. What is the image used by the deployment?
kubectl describe deployment web-dashboard

- Inspect the Dashboard Application POD and identify the Service Account mounted on it?
kubectl describe pod web-dashboard-6cbbc88b59-r2pnl

- Create a new ServiceAccount named dashboard-sa?
kubectl create serviceaccount dashboard-sa

```

#### Resource Requirements
- Kubernetes assumes that a Pod or a container within a pod requires 0.5 CPU and 256 mebibyte(Mi) of memory.
- Limits and requests are set to each container within the pod
- When a pod tries to exceed resources beyond its specified limit
  - CPU: Kubernetes throttles
  - Memory: A container can use more memory than its limit, but if the container tries to consume more memory than its limit constantly the pod will be terminated
```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-dashboard
      image: my-dashboard
      ports:
      - containerPort: 8080
      resources:
        requests:
          memory: "1Gi"
          cpu: 1
        limits:
          memory: "2Gi"
          cpu: 2
```

##### Questions - Resource Limits
```cmd
- A pod called rabbit is deployed. Identify the CPU requirements set on the Pod
kubectl describe pod rabbit
```

#### Taints and Tolerations
- Taints and Toleration are used to set restrictions on what pods can be scheduled on a node
- If we apply a toleration on a pod, and a taint on a node, that specific pod can only be deployed in that node because of the toleration
- Taints are set on nodes while Tolerations are set on pods
- There are 3 taint-effects
  1. NoSchedule: Pods will not be scheduled on the node
  2. PreferNoSchedule: The system will try to avoid placing a pod on the node
  3. NoExecute: New pods will not be scheduled on the node, if there are any existing pods on the node that do not tolerate the taint will be evicted
- Master node have a taint to stop pods from being scheduled on the node itself

```cmd
- Apply a Taint on a node
kubectl taint nodes $node_name key=value:$taint_effect
kubectl taint nodes node1 app=blue:NoSchedule

- Check master node taint
kubectl describe node kubemaster | grep Taint
```
- Add a Toleration to a Pod
```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"
```

##### Questions - Taints and Tolerations
```cmd
- How many nodes exist on the system?
kubectl get nodes

- Do any taints exist on node01 node?
kubectl describe node node01 | grep Taints

- Create a taint on node01 with key of spray, value of mortein and effect of NoSchedule
kubectl taint nodes node01 spray=mortein:NoSchedule

- Create a new pod with the nginx image and pod name as mosquito
kubectl run mosquito --image=ngin

- Do you see any taints on controlplane node?
kubectl describe node controlplane | grep Taint

- Remove the taint on controlplane, which currently has the taint effect of NoSchedule.
kubectl edit node controlplane
```

#### Node Selectors
- We can define some limitations on the pod so that they can only run on particular nodes
- key=value labels are assigned to the node, scheduler uses these labels to match and identify the right node to place the pods on
- There are some limitations with NodeSelectors, for that we have NodeAffinity and NodeAntiAffnity rules

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  nodeSelector:
    size: Large
```

```cmd
- Labeling a node
kubectl label node $node_name $key=$value
```

#### NodeAffinity
- To make sure that pods are hosted on particular nodes

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In # NotIn Exists
            values:
            - Large
            - Medium
```

- Node Affinity Types
  1. requiredDuringSchedulingIgnoredDuringExecution: If these rules are matching only pods are placed on nodes
  2. preferredDuringSchedulingIgnoredDuringExecution: If there are no matching rules pods might schedule on any node
  3. requiredDuringSchedulingRequiredDuringExecution
   
##### Questions - Node Affinity
```cmd
- How many Labels exist on node node01?
kubectl get nodes node01 --show-labels

- Apply a label color=blue to node node01?
kubectl label node node01 color=blue

- Create a new deployment named blue with the nginx image and 3 replicas?
kubectl create deployment blue --image=nginx --replicas=3
```

#### Taints Tolerations and Node Affinity

- We have Blue, Red, Green and Other nodes. We have blue, red, green and other pods as well. Each separate pod must reside in the correct node. It should not be scheduled in a different node.
- If use use both taints and tolerations that will not make sure that respective pods will not ends up in a different node which doesnt have any taints at all. It will make sure if there is a taint on the node, only tolerable pods are placed upon the node.
- If we use node affinity to label each nodes and then set nodeSelectors on the pods to tie them to their nodes. That will not make sure that other pods will be placed on these nodes.
- For this to happen, we can use both Taint and Tolerations along with node affinity
- We use Taints and Tolerations to stop other pods from placed on our nodes, then use node affinity to prevent our pods from being placed on their nodes.

#### Multi Container Pods
- Created together, destroyed together to share the same lifecycle.
- They have access to each other through localhost and they have access to the same storage volumes

- Design Patterns
  1. SideCar Pattern - Log agent
  2. Adapter Pattern - Centralized agent to convert the messages to a common format
  3. Ambassador Pattern - Choosing the correct database but being able to use localhost throughout the application code

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
  - name: log-agent
    image: log-agent
```

##### Questions - Multi Container Pods
```cmd
- Identify the number of containers created in the red pod?
kubectl describe pod red

- Inspect the app pod and identify the number of containers in it. It is deployed in the elastic-stack namespace?
kubectl describe pod app -n elastic-stack

- The application outputs logs to the file /log/app.log. View the logs and try to identify the user having issues with Login?
kubectl logs app
```

#### InitContainers

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ;']
```

##### Questions - InitContainers
```cmd
- Identify the pod that has an initContainer configured.
kubectl describe pod blue
```

#### Readiness and Liveness Probes
- Pod status can be from Pending, ContainerCreating and Running
- We need to tie the ready condition to the actual state of the application inside the container

- Readiness Probes
  1. HTTP Test - Testing an API

  ```yml
  readinessProbe:
    httpGet:
      path: /api/ready
      port: 8080
    initialDelaySeconds: 10
    periodSeconds: 5
    failureThreshold: 8
  ```

  2. TCP Test - Testing a port available

  ```yml
  readinessProbe:
    tcpSocket:
      port: 3306
  ```

  3. Exec Command - Running a script inside the container

  ```yml
  readinessProbe:
    exec:
      command: ["cat", "/app/is_ready"]
  ```

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: myapp
    ports:
    - containerPort: 8080
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
```

- Liveness Probes
- What if the container is up, but due to a bug application is stuck in an infinite loop and hence not working.
- Liveness probe can be configured on the container to periodically test whether the application within the contianer is actually healthy. If the test fails, the container is considered unhealthy and destryed and recreated.

  1. HTTP Test - Testing an API

  ```yml
  livenessProbe:
    httpGet:
      path: /api/ready
      port: 8080
    initialDelaySeconds: 10
    periodSeconds: 5
    failureThreshold: 8
  ```

  2. TCP Test - Testing a port available

  ```yml
  livenessProbe:
    tcpSocket:
      port: 3306
  ```

  3. Exec Command - Running a script inside the container

  ```yml
  livenessProbe:
    exec:
      command: ["cat", "/app/is_ready"]
  ```

#### Logging

```cmd
- Create a pod
kubectl create -f event-simulator.yml

- Trail the logs
kubectl logs -f event-simulator-pod

- When there are multiple containers
kubectl logs -f event-simulator-pod image-processor
```

#### Monitoring- Metrics Server
- You can have 1 metrics server per kubernetes cluster
- Metrics server retrieves metrics from each of the kubernetes nodes and pods, aggregate them and stores them in memory
- Kubernetes runs an agent on each node, known as the kubelet, which is responsible for receiving instructions from the kubernetes API master server
- Kubelet contains a sub component known as cAdvisor or Container Advisor
- CAdvisor is responsible for retrieving performance metrics from pods and exposing them through the kubelet API to make the metrics available for the metrics server

```cmd
- Run below command to run metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

##### Questions - Monitoring
```cmd
- Identify the node that consumes the most Memory(bytes).
kubectl top node
```

#### Labels, Selectors and Annotations

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: App1
    function: Front-end
spec:
  containers:
  - name: myapp-container
    image: myapp
    ports:
    - containerPort: 8080
```

```cmd
- Getting all the pods with specific label
kubectl get pods --selector app=App1
```

```yml
apiVersion: v1
kind: ReplicaSet
metadata: 
  name: myapp-rs
  labels: 
    app: App1
    function: Front-end
spec: 
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      name: myapp-pod
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
```

##### Questions - Labels, Selectors and Annotations
```cmd
- We have deployed a number of PODs. They are labelled with tier, env and bu. How many PODs exist in the dev environment (env)?
kubectl get pods --selector env=dev

- How many PODs are in the finance business unit (bu)?
kubectl get pods --selector bu=finance

- How many objects are in the prod environment including PODs, ReplicaSets and any other objects?
kubectl get all --selector env=prod

- Identify the POD which is part of the prod environment, the finance BU and of frontend tier?
kubectl get pods --selector env=prod,bu=finance,tier=frontend
```

#### Rolling Updates and RollBacks
- When you created a deployment a new rollout is triggered with a new revision number
- 2 strategies
  1. Recreate: Destroys all the older pods and create new pods
  2. Rolling update: default, take down older version and bring up a newer version one by one
```cmd
- Check the status of a rollout
kubectl rollout status $deployment_name

- Show the history of the revisions
kubectl rollout history $deployment_name

kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1

- Rollback to previous revision
kubectl rollout undo $deployment_name
```

##### Updates and Rollbacks
```cmd
- Create a nginx deployment with 3 replicas
kubectl create deployment nginx --image=nginx --replicas=3 --dry-run=client -o yaml > mydeployment.yml

- Apply the deployment
kubectl apply -f mydeployment.yml --record

- Check the status of the deployment
kubectl rollout status deployment/nginx

- Check the history of revisions
kubectl rollout history deployment/nginx

- Delete the deployment
kubectl delete deployment nginx

- Changing the version using set image
kubectl set image deployment/nginx nginx=ngnix:1.12

- Rollback to previous revision
kubectl rollout undo deployment/nginx
```

##### Questions - Updates and Rollbacks
```cmd
- What container image is used to deploy the applications?
kubectl describe deployment frontend

- Upgrade the application by setting the image on the deployment to kodekloud/webapp-color:v2
kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v2
```

#### Jobs

- To stop the pod from restarting after exiting
```yml
apiVersion: v1
kind: Pod
metadata: 
  name: math-pod
spec: 
  containers:
    - name: math-add
      image: ubuntu
      command: ["expr", "3", "+", "2"]
  restartPolicy: Never
```

- Job definition file
```yml
apiVersion: batch/v1
kind: Job
metadata: 
  name: math-add-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec: 
      containers:
        - name: math-add
          image: ubuntu
          command: ["expr", "3", "+", "2"]
      restartPolicy: Never
```

```cmd
kubectl get jobs
kubectl get pods
kubectl logs $pod_name
kubectl delete job $pod_name
```

#### CronJobs

```yml
apiVersion: batch/v1beta1
kind: CronJob
metadata: 
  name: report-cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      template:
        spec: 
          containers:
            - name: math-add
              image: ubuntu
              command: ["expr", "3", "+", "2"]
          restartPolicy: Never
```

##### Questions - Jobs and CronJobs
```cmd
- Let us now schedule that job to run at 21:30 hours every day.
kubectl create cronjob throw-dice-cron-job --image=kodekloud/throw-dice --schedule='30 21 * * *'
```

#### Services
- There are multiple services
  1. NodePort: Service makes an internal port accessible on a port on the node
  - There are 3 ports involved in this service
    1. targetPort - Port on the pod
    2. port - Port on the service
    3. nodePort - Port on the node(30000-32767)

```yml
apiVersion: v1
kind: Service
metadata: 
  name: myapp-service
spec: 
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

  2. ClusterIP: Service creates a virtual IP inside the cluster to enable communication between different services

```yml
apiVersion: v1
kind: Service
metadata: 
  name: myapp-service
spec: 
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
    type: front-end
```

  3. LoadBalancer: Provisions a load balancer for our application in supported cloud provider

##### Questions - Services
```cmd
- How many Services exist on the system?
kubectl get svc

- What is the targetPort configured on the kubernetes service?
kubectl describe svc kubernetes
```

#### Ingress
- Ingress helps your users access your application using a single external accessible URL that youcan configure to route traffic to different services within your cluster. Can implement SSL security as well
- We can create ingress resources just like deployments, pods, services etc
- Ingress controller is not deployed to kubernetes cluster by default
- ex:
  1. GCE
  2. Nginx
- These ingress controllers are not just another load balancer or nginx server, the load balancer component are just a part of it
- Ingress controllers have additional intelligence built into them to monitor the kubernetes cluster for new definitions or ingress resources

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-service
            port:
              number: 80
      - path: /watch
        pathType: Prefix
        backend:
          service:
            name: watch-service
            port:
              number: 80
```

```cmd
kubectl create -f ingress-wear.yml
kubectl get ingress
kubectl describe ingress $ingress_name
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80" --dry-run=client -o yaml > ingress.yml
```

##### Questions - Ingress Networking 1
```cmd
- Which namespace is the Ingress Controller deployed in?
kubectl get all -A

- Which namespace are the applications deployed in?
kubectl describe deploy ingress-nginx-controller --namespace ingress-nginx

- What is the name of the Ingress Resource?
kubectl get ingress --namespace app-space

- A new payment service has been introduced. Since it is critical, the new application is deployed in its own namespace.
 kubectl get svc -A -o wide

- You are requested to make the new application available at /pay.
kubectl create ingress ingress-pay -n critical-space --rule="/pay=pay-service:8282"
```

##### Questions - Ingress Networking 2
```cmd
- We have deployed two applications. Explore the setup.
kubectl get all -A

Let us now deploy an Ingress Controller. First, create a namespace called ingress-nginx
kubectl create namespace ingress-nginx

- The NGINX Ingress Controller requires a ConfigMap object. Create a ConfigMap object with name ingress-nginx-controller in the ingress-nginx namespace.
kubectl create configmap ingress-nginx-controller -n ingress-nginx

- The NGINX Ingress Controller requires two ServiceAccounts. Create both ServiceAccount with name ingress-nginx and ingress-nginx-admission in the ingress-nginx namespace.
kubectl create serviceaccount ingress-nginx -n ingress-nginx
kubectl create serviceaccount ingress-nginx-admission -n ingress-nginx

- Create the ingress resource to make the applications available at /wear and /watch on the Ingress service.
kubectl create ingress ingress-wear-watch -n app-space --rule="/wear=wear-service:8080" --rule="/watch=video-service:8080"
```

#### Network Policies
- All the pods inside the kubernetes cluster can communicate with each other
- We can use a network policy to allow/deny communication with each other
- Below definition is for the db-pod to allow traffic from api-pod through port 3306

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
        matchLabels:
          name: prod
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80  
```
- In above yml we add a network policy and use labels and selectors to associate the policy with the pods.
- `role: db` are the labels to match the database pods
- We need to make sure only api-pod can communicate to db-pod but only through port 3306
- If we haven't specify a policy type this wont deny traffic to those pods, for that policyTypes is mandatory
- Once you allow ingress traffic you dont need to specify a separate rule for egress traffic as well
- You only need to add the traffic which is originating from source to target

##### Questions - Network Policies
```cmd
- What is the name of the Network Policy?
kubectl get networkpolicies

- Create a network policy to allow traffic from the Internal application only to the payroll-service and db-service? Policy Name internal-policy;Policy Type: Egress;Egress Allow: payroll;ayroll Port: 8080;Egress Allow: mysql;MySQL Port: 3306
kubectl apply -f internal.yml
```

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - port: 8080
      protocol: TCP
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - port: 3306
      protocol: TCP
```

#### Volumes & Mounts

```yml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    
    volumeMounts:
    - mountPAth: /opt
      name: data-volume
      
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
```

#### Persistent Volumes

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

```cmd
kubectl create -f pv-definition.yml
kubectl get persistentvolume
```

#### Persisten Volume Claims

- Every Persistent Volume Claim is bound to a single Persistent Volume(1:1)
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myClaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

#### Storage Classes
- Static provisioning
  - Before creating any Persistent Volume we need to manually create the relevant disk 
- Dynamic provisioning
  - Volume gets provisioned automatically when the application needs it. You can use Storage classes for this

```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/
```

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myClaim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: 500Mi
```

```yml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    
    volumeMounts:
    - mountPAth: /opt
      name: data-volume
      
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: myClaim
```

##### Questions - Storage Class
```cmd
- How many StorageClasses exist in the cluster right now?
kubectl get sc
```

#### Stateful Sets
- If the instances need a particular order and a name you can use statefulsets
- podManagementPolicy: Parallel: add this to make the statefulset not follow the ordered approach(default value is OrderedReady)
```yml
apiVersion: apps/v1
kind: StatefulSet
metadata: 
  name: mysql
  labels: 
    app: mysql
spec: 
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  serviceName: mysql-h
  podManagementPolicy: Parallel
```

```cmd
kubectl create -f statefulset-definition.yml
kubectl scale statefulset mysql --replicase=5
kubectl scale statefulset mysql --replicase=3
kubectl delete statefulset mysql
```

#### Headless Services

- We create a service so that the web application can talk to database server using the service and all the requests are load balanced across all the database pods in the deployment.
- We need a service that doesnt load balance requests but gives us a DNS entry to reach each pod. Thats a headless service.
- When you create a headless service all the DNS names are created in following manner
  - eg: podname.headless-servicename.namespace.svc.cluster.local
        mysql-0.mysql-h.default.svc.cluster.local -> master
        mysql-1.mysql-h.default.svc.cluster.local -> slave1
        mysql-2.mysql-h.default.svc.cluster.local -> slave2
        
```yml
apiVersion: v1
kind: Service
metadata:
  name: mysql-h
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None
```

- You must define subdomain value to the name of the service name, so that it will create DNS entries for the name of the service to point to the pod
- To create A records you need to specify hostname
- By default in a deployment file, if there are no values for subdomain and hostname, a headless service will not create A record for the pod
- If we add the pod defintion in a deployment all the pods will get the same A record mysql-pod.mysql-h.default.svc.cluster.local and this will not help us to meet of addressing the pods separately
- To overcome the issue we can use the StatefulSet and we need to explicitly define the serviceName so that it can identify the headless service

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: mysql
spec:
  containers:
  - name: mysql
    image: mysql
  subdomain: mysql-h
  hostname: mysql-pod
```

#### Storage and StatefulSet
- If we need underlying all pods to share the same database storage we can do that. But for each pod to have a seaparate storage? 
- Then each pod needs a PVC that bounds to a PV. These PVs can be created from a single SC or multiple SC
- Instead of maintaining a separate PVC-definition we can move the content to VolumeClaimTemplates in statefulset-definition
- StatefulSets doesnt automatically deletes PVC when a pod gets recreated or rescheduled in the same node or a different node, it ensures that the pod is reattached to the same PVC it was attached before. Thus StatefulSets guarantees statusful storage for pods


#### Dockerfile
- Dockerfile contains instructions and arguments

```Dockerfile
FROM ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

##### Questions - Docker images
```cmd
- Build a docker image using the Dockerfile and name it webapp-color. No tag to be specified?
docker build -t webapp-color .

- What is the base Operating System used by the python:3.6 image?
docker history python:3.6
```

#### Authentication and Authorization
- kube-apiserver is at the center of all opearions within kubernetes.
- We need to see who can access the api-server and what can they do
- We can access using
  1. Username and Passwords
  2. Username and Tokens
  3. Certificates
  4. LDAP Authentication providers
  5. Service Accounts

- Authroization is implemented using
  1. RBAC Authorizarion
  2. ABAC Authorization
  3. Node Authorization
  4. Webhook mode

#### Authentication
- Mainly Users(Admins and Developers) and ServiceAccounts
- All user access is managed by the API server
- Whether you are accessing the cluster through kubectl tool or the API directly all these requests go through the kube-apiserver and it authenticates the requests before processing it

1. Username Password mechanism
- Create a user-details.csv where you have a list of users with their password, name and userid
- Add it to kube-apiserver.service file as below `--basic-auth-file=user-details.csv`

2. User Token mechanism
- Create a user-token-details.csv where you have a list of users with their tokens, name, userid and groupid
- Add it to kube-apiserver.service file as below `--token-auth-file=user-token-details.csv`

#### KubeConfig

```cmd
kubectl get pods 
                --server $server_address 
                --client-key admin.key 
                --client-certificate admin.crt 
                --certificate-authority ca.crt
```
- Typing these commands everytime is a tedious task, so we move these to a configuration file called kubeconfig

```cmd
kubectl get pods --kubeconfig config

// View the current config
kubectl config view

// View the given config
kubectl config view --kubeconfig=my-custom-config

// Change the current context
kubectl config use-context user@prod 
```

- If you create the config file in $HOME/.kube/config location(which is the default path for the config file) you dont need to explicitly add the --kubeconfig config in above command as well
- config file has 3 sections
  - clusters - varies k8s clusters that you have access to (in above --server)
  - Users - User accounts which have access to these clusters (in above except --server belongs to here)
  - Contexts - Binds which user account is avaiable for which cluster

```yml
apiVersion: v1
kind: Config

current-context: my-kube-admin@my-kube-playground

clusters:
- name: my-kube-playground
  cluster:
    certificate-authority: ca.crt
    server: https://my-kube-playground:6443

contexts:
- name: my-kube-admin@my-kube-playground
  context:
    cluster: my-kube-playground
    user: my-kube-admin

users:
- name: my-kube-admin
  user:
    client-certificate: admin.crt
    client-key: admin.key
```

##### Questions - KubeConfig
```cmd
- Where is the default kubeconfig file located in the current environment?
$HOME/.kube/config

- How many clusters are defined in the default kubeconfig file?
kubectl config view

- A new kubeconfig file named my-kube-config is created. It is placed in the /root directory. How many clusters are defined in that kubeconfig file?
kubectl config view --kubeconfig=my-kube-config

- I would like to use the dev-user to access test-cluster-1. Set the current context to the right one so I can do that?
kubectl config --kubeconfig=my-kube-config use-context research
```

#### API Groups
- APIs are categorized into 2 groups
  - Core group: /api
    - All the core functionality is maintained. 
      - eg: namespaces, pods, rc, nodes, PV, PVC, configmaps, secrets, services
  - Named group: /apis
    - More organized and all the newer features are going to be made available through these named groups
      - eg: /apps, /extensions, /networking.k8s.io, /storage.k8s.io, /authentication.k8s.io, /certificates.k8s.io


```cmd
curl http://localhost:6443 -k

curl http://localhost:6443/apis -k | grep "name"

kubectl proxy
// starts a proxy server on port 8001 locally and user credentials and certificates from your kubeconfig file to access the cluster

curl http://localhost:8001 -k
So the proxy will use the credentials from the kubeconfig file to forward the request to the kube API server
```

#### Authorization
- Below are the different way to of authorizing in Kubernetes
  - Node
  - ABAC: Attribute Bases Authorization
    - You need to edit the policy file everytime you need to make change and restart the kube API server
  - RBAC: Role Based Authorization
  - Webhook