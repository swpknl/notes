- It is a container orchestration framework
- When we install Kubernetes in a system, we install the following:
	- API server - acts as the frontend for Kubernetes, all clients talk to the API server
	- etcd - distributed, reliable key-value store to store all data related to Kubernetes. Responsible for implementing logs
	- kubelet - agent that runs on each nodes in the cluster
	- Container runtime - Responsible for running the container, eg Docker
	- Controller - this is the brain behind orchestration. Brings up new containers when one ends
	- Scheduler - responsible for distributing loads/work
- We have master and worker servers. Master is the orchestrator
- Workers are where the containers are hosted
- The master server has the Kube API server
- Workers have the kubelet agent which talks to the Kube API server
- All information created is stored in etcd in master
- Master also has controller and scheduler
- There are is kube command line tool, or kubectl or kube control tool. 

Docker vs containerd
	- OCI - Open Container Initiative
		- Defined image spec and runtime spec
	- Kubernetes and Docker were tightly coupled
	- To decouple and allow other container runtimes, Kubernetes created CRI (Container Runtime Interface)
	- Docker does not support CRI, so a dockershim was created to support Kubernetes
	- containerd is CRI compatible and can be used with Kubernetes
	- In v1.24 of Kubernetes, support for dockershim was removed and Docker was removed as a supported runtime
	- ctr is the CLI for containerd - not very user friendly and only supports limited features
	- nerdctl is a Docker-like CLI for containerd
	- It is similar to Docker cli
	- crictl - provides a CLI for CRI compatible container runtimes
		- Installed separately
		- Developed and maintained by the Kubernetes community

Setting up Kubernetes
- Can be setup locally using minikube, microk8s
- Hosted solutions available in cloud, like in GCP, Azure etc

Minikube
- minikube bundles master and node together in a single image to run a single node cluster

Pod
- Pod is the smallest object you can create in Kubernetes
- It is a single instance of an application
- We create a new pod in a node when we want to scale horizontally
- If current capacity is full, add a new node in a cluster and add new pods in the new node.
- To scale down, delete existing pod
- A single pod can have multiple container, but they are not of the same kind

kubectl	
```bash
kubectl run nginx --image nginx
```

- Image is downloaded from Docker hub. Image is name in dockerhub and name here is nginix
- Create a pod in a node with the nginx image
```bash
kubectl get pods
```
- Shows the list of pods in a cluster
```bash
kubectl describe pod nginx
```
- provides more detailed info than get pods
```bash
kubectl get pods -o wide #(additional options)
```
- Gives a fairly detailed info on pods

Pods using yaml:
- Kubernetes yaml file always contains 4 top level/root level required fields in its yaml file:
	- apiVersion: version of K8s api
	- kind: the kind of K8s object to be created. Other values, Pod, service, ReplicateSet, Deployment. It is case sensitive
	- metadata: Data about the object, like name, label. It is a dictionary
	- spec: Provide additional information about the object being created. It is a dictionary
- Pods can have multiple containers in it
Command to use yml file: 
```bash
kubectl create -f pod-definition.yml
```
Apply an edit command after applying: 
```bash
kubectl apply -f pod-definition.yaml
```
Use the command to get information about a pod:
```bash
kubectl get pods
```
To get a detailed view of a pod, use:
```bash
kubectl describe pod myapp-pod
```
To create a pod using command line (without file), use:
```bash
kubectl run my-pod --image=nginx
```
View all pods in detail:
```bash
kubectl get pods -o wide
```

Replication controller
Replication controller helps in running multiple instances of a single pod in a node
It'll help in brining up a new pod when the current one fails
It spans across multiple nodes across the cluster
It helps balance the load across multiple pods across nodes

RelplicaSet
Both have the same purpose, but ReplicationController is the old technology, and is replaced by ReplicaSet

rc-definition.yml -> this is for ReplicaController
```yaml
apiVersion: v1
kind: ReplicationController
metadata: 
	- name: myapp-rc
	- labels:
		- app: my-app
		- type: front-end
- spec: # this is the most important section, it defines what is inside the object that it is creating
	template: # Provide a pod template to create replicas
		metadata:
			name: myapp-pod
			type: front-end
		spec:
			containers:
				name: ngnix-container
				image: nginx
	replicas: 3 # same level as template
```


re-definition.yml => this is for ReplicateSet
- apiVersion: apps/v1 <- this is important, ReplicaSet is not present in v1
- apart from ReplicationController, it has selector
- selector: <- it defines what pods fall under it. It is the major difference between RC and RS

- selector:
	- matchLabels: <- match the labels to control
		- type: front-end

Labels and selectors:
Role of RS: process that monitors the pods
If any one of them fails, then it creates a new one to maintain the desired number of pods
How does it identify which pod to control across various pods:
Using labels
Scaling replicasets:
- We can update the number of replicas in the yaml definition file
```bash
kubectl create -f defintion.yml
```

```bash
kubectl scale --replicas=6 -f definition-file.yml
```
- This will not be updated in the file
```bash
kubectl scale --replicas=6 replicaset myapp-replicaset
```

We can also expand replicas based on load
We can also edit the replicaset using the following command:
kubectl edit replicateset name-of-replicateset

It opens up in vim, which is the running configuration, and not the actual file which was created. It is a temporary file created by K8s and has other details

Scale replicas:
```bash
kubectl scale replicaset app-name --replicas=2
```

Delete pod:
```bash
kubectl delete pod pod-name
```

Explain syntax:
```bash
kubectl explain replicaset
```

Deployments:
Handles deployments for K8s
```bash
kubectl create -f filename.yaml
kubectl get deployments
kubectl get all 
```

Rollout and versioning:
When first deployment triggers a rollout. A new rollout creates a revision
In the future when the application is upgraded, a new rollout is created, which triggers a new revision
this allows us to upgrade/downgrade as required

kubectl rollout history deployment-name

Deployment strategy:
- Destroy current instances and then deploy new versions
	- Downtime between destruction and new deploymenty
	- Called as Recreate strategy
- We take down older version and bring up newer version one by one
	- it is seamless
	- this is called as rolling update and is the default deployment strategy

Edit the file and apply the changes
```bash
kubectl apply -f file_name
```

Another way is to:
```bash
kubectl set image deployment-name image version
```
This does not update the definition file, so must be used carefully

Edit inline:
```bash
kubectl edit deployment frontend # Notes: this will open vim
```

Rollback:
```bash
kubectl rollout undo deployment-name
```

Rollout status:
```bash
kubectl rollout status deployment-name
```

Rollout history:
```bash
kubectl rollout history deployment-name
```

Add record to record changes
```bash
kubectl create -f deployment-definition.yml --record # This records the change-cause in the kubectl rollout 
```

If for some reason there is a mistake in the definition.yml file, like wrong image name etc, and when we try to deploy this deployment, K8s will proactively stop terminating the old pods if when starting the new pods we get an error.

Networking:
Each pod is given an IP address, not a container
Accessing sibling pods via IP is not recommended, as the IP may change when the pod is recreated
K8s networking requirements:
- Pods must be able to communicate with each other without NAT

We can use a managed provider to handle cluster networking. There are various providers available for the same

Kubernetes Services:
- It is another object like ReplicaSet, Pod etc
	- Eg. Node Port service, which makes the internal service accessible to the external internet
- Types:
	- Node port
	- Load balancer
	- Cluster IP

	- Node Port:
		- Consists of:
			- Target port
			- port of the service. IP of the node is via cluster IP
			- Port on the node, known as Node port
				- The port range is from 30000 to 32767
	- Spec file for Node port:
``` yaml
apiVersion: v1
kind: Service
metadata:
	name: node-port
spec:
	type: NodePort
	ports:
		- targetPort: 80
		  port: 80
		  nodePort: 30008
	- selector: # list of labels to identify the pod, it'll select all pods matching the selector. Works with distributed nodes. A service is created that spans multiple nodes across clusters and the config is created for each pod in the node
		app: app-type
		type: front-end	 
```

- port is manadatory
- if targetport is not given, then the value is that of port
- if nodePort is not given, then the value is given as any free port available


ClusterIP:
- Contains only port and targetPort in its spec file
- Type is ClusterIP'

LoadBalancer:
- LoadBalancer on supported cloud balancer will work with the following:
	- Type: LoadBalancer
	- Spec:
		- ports containing port, nodePort and targetPort

Docker run linking:
- Use the --name option to name a container
- This name is then used to link containers together
```bash
docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
```
- --link follows hostName:containerName format
- This creates an entry in the etc/hosts file with the internal IP of the linked container


Same setup in K8s:
- Deploy pods
- Create services (ClusterIP)
	- redis
	- db
- Create services (NodePort)
	- voting-app
	- result-app