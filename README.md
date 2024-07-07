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

- What is the image used to create the new pods?
kubectl describe pod newpods-92tl5

- Which nodes are these pods placed on?
kubectl get pods -o wide

- What images are used in the new webapp pod?
kubectl describe pod webapp

- Delete the webapp Pod
kubectl delete pod webapp
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
```text
db-service   . dev       . svc     . cluster.local
Service Name   Namespace   Service   Domain   
```

```cmd
- Get pods in a different namespace
kubectl get pods --namespace=kube-system

- Create a namespace
kubectl create namespace $namespace_name

- Set the namespace to 'dev'
kubectl config set-context $(kubectl config current-context) --namespace=dev

- Get pods in all the namespaces
kubectl get pods --all-namespaces
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
kubectl get pods --namespace=marketing
kubectl get pods --all-namespaces
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

##### Questions - Imperative Commands
```cmd
- Deploy a pod named nginx-pod using the nginx:alpine image?
kubectl run nginx-pod --image=nginx:alpine

- Deploy a redis pod using the redis:alpine image with the labels set to tier=db?
kubectl run redis --image=redis:alpine --labels="tier=db"

- Create a service redis-service to expose the redis application within the cluster on port 6379?
kubectl create svc clusterip --tcp=6379:6379 redis-service

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