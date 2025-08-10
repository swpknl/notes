Node: It is a worker machine where containers will be launched by K8s
Cluster: collection of nodes grouped together
Master: Stores info about the cluster, nodes, failed nodes etc and takes action

API Server - frontend for K8s
etcd - Reliable key-value pair to store data about the cluster
kubelet - agent that runs on each node in the cluster
Container runtime - underlying runtime to run container
Controller - 
Scheduler - responsible for distributing work across nodes

Master server has the kube-apiserver which makes it the master

Helper containers can be of the same pod, and share the same lifecycle
ReplicaSet spec section:
	- template - required to specify to the replicaSet how to create a new pod once a current one fails
	- replicas
	- selector
		- matchLabels

Namespace:
db-service.dev.svc.cluster.local
cluster.local - domain of the local
svc - service
dev - namespace
db-service - name of the service

```bash
kubectl get pods --namespace=dev
```

```bash
kubectl create -f pod-definition.yml --namespace=dev # namespace=dev under metadata in pod-definition.yml
```

Namespace:
apiVersion: v1
kind: Namespace
metadata:
	name: dev

```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

Limit resources in a namespace:
apiVersion: v1
kind: ResourceQuota


```bash
kubectl expose pod redis --port 6379 --name redis-service
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
kubectl run custom-nginx --image=nginx --port=8080
kubectl create namespace dev-ns
```

Command overrides entrypoint in dockerfile
args field overrides the cmd field in dockerfile
use env to use environment variable
env:
	- name: AAS
	   value: 1213

Two phases in creating configMap:
- Create configMap
- Inject into pod

Two ways of creating configMap:
- Imperative
```bash
kubectl create configmap name --from-literal=key=value
kubectl craete configmap name --from-file=path.properties
```
- Declarative
```yaml
apiVersion: v1
kind: ConfigMap
metadata: 
	name: app-config
data:
	- APP_COLOR: blue
```

```bash
kubectl get configmaps
```

Inject:
```yaml
spec:
	containers:
		- name: simple-webapp-color
		- envFrom:
			- configMapRef:
				name: app-config
```

Secret:
1. Create one
2. inject it into pod

Create usoing:
- Imperative 
```bash
kubectl create secret generic <secret-name> --from-literal=<key>=<value>
kubectl create secret generic <secret-name> --from-file  fileName.yaml
```
- Declarative
```yaml
apiVersion: v1
kind: Secret
metadata:
	name: app-secret
data:
	DB_HOST: mysql
```
specify in encoded format in base64 format
echo -n 'mysql' | base64

To display secret:
```bash
kubectl get secret app-secret -o yaml
```
view decoded:
```bash
echo -n 'mysql' | base64 --decoded
```
- secrets are not encrypted, only encoded

```yaml
envFrom:
  - configMapRef:
      name: my-config

envFrom:
  - secretRef:
      name: my-secret
```

```yaml
securityContext:
	runAsUser: 1000
	 capabilities:
		 add: ["MAC_ADMIN"]
```

```yaml
resources:
	limits:
		cpu:
		 memory:
	 requests:
		 cpu:
		 memory:
```

Service account:

Taint and tolerance:
```bash
kubectl taint nodes node01 spray=mortein:NoSchedule
```

Remove taint: (add -)
```bash
kubectl taint nodes node01 spray=mortein:NoSchedule-
```

nodeSelector:
	size: large

```yaml
kubectl label nodes node01 color=blue
```

Multicontainer design patterns:
- Co-located containers
- Regular init containers
- Sidecar containers: Setup like an init container, but instead of ending like an init container, it continues to run after initialization

```yaml
spec:
	containers:
		- name:
		   image:
		  ports:
				containerPort:
		 initContainers:
		 - name:
	       image:
	       command:
	       restartPolicy:
```

Check k8s logs:
```bash
kubectl logs pod-name -n namespace
```

Execute inside a container:
```bash
kubectl exec -it pod-name -n namespace -- cat/logs
```

Pod lifecycle:
- Pending - where to place the pod in the node
- Scheduled
- ContainerCreating 
- Running

Conditions:
- PodScheduled
- Initialized
- ContainersReady
- Ready


readinessProbe/livenessProbe:
```yaml
httpGet:
		port: 8080
		 path: /api/ready
	 tcpSocket:
		 port: 3306
	 exec:
		 command:
			 - cat
			 - /app/is_ready
```
			
```bash
kubectl logs -f pod-name container_name # -f streams live logs
```
incase multiple containers are present, provide container name

```bash
kubectl top node
kubectl top pod

kubectl get pods --selector app=App1,tier=Frontend

k scale deployment --replicas=2 frontend
```

K8s job:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
	name: math-add-job
spec:
	completions: 3
	template:
		spec:
			containers:
				- name: math-add
				  image: ubuntu
```


```bash
k create -f job-definition.yaml

k get jobs
k get pods
k delete job math-add-job
```

Jobs are created till completion value is realised
Add parallelism: 3 to create parallely

CronJobs:
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
	name: reporting
spec:
	schedule: "*/1 * * * *"
	jobTemplate:
		spec:
			completions: 3
			 parallelism: 3
			 template:
				 spec:
					 containers:
						 - name: reporting-tool
```

NodePort - service:
- TargetPort - port in pod
- Port - port in service(NodePort). IP of Cluster is called ClusterIP
- NodePort - port in node (Range 30000 - 32767)


K8s has an All allow traffic rule by default
Network policy is linked to one or more pods. To restrict traffic to pods

```yaml
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

Docker stores data in /var/lib/docker
If commands are same, then docker will reuse the layers
When source code changes, docker reuses the layers and only changes the source code file
It creates a writeable layer for the docker image to store the temp files like logs etc
The layers built using the Dockerfile are readonly
If the container is deleted, all the files in the read write layer are also deleted
Persist data:
docker volume create volume_data
```
/var/lib/docker
	/volumes
		/data_volume
```

```bash
docker run -v data_volume:/var/lib/mysql mysql # -v creates a new volume automatically if not already not created
```

We can provide another folder as well to mount to docker: (also known as bind mount)
```bash
docker run -v /data/mysql:/var/lib/mysql mysql
```

-v is the older way
new way is -mount
```bash
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```
These are used using storage drivers

There are also volume driver plugins:
```yaml
spec:
	containers:

	 volumeMounts:
		 - name: data-volume
		   mountPath: /opt
	 
	 volumes:
	 - name: data-volume
	   host-path:
			 path: /data
			 type: Directory
```

Persistent volume:
```yaml
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

Security:
- Can be controlled via access to the kube-apiserver
- All pods can access all other pods within the server

Authentication mechanism:
- Static password file:
	- csv file with password, username, userid
	- Has to be set in the kube-apiserver.yaml
- Static token file:
	- static token instead of password

Both these mechanisms are insecure and should not be used and has been deprecated in v1.19

Use cert with curl. Same with password and token

To use with kubectl:
create kubeconfig file in $HOME/.kube/config is the default location of kubeconfig
```bash
kubectl get pods --kubeconfig kubeconfigFile
```

Kubeconfig file:
```yaml
apiVersion: v1
kind: Config
current-context: dev-user@google
clusters:
- name: y-kube-playground
  cluster:
		certificate-authority: 
		 server:
contexts:
- name:

users:
- name: my-kube-admin
```


```bash
kubectl config view
kubectl config use-context prod-user@production

kubectl auth can-i create deployments
kubectl auth can-i --as dev-user get pods
```

Get authorization modes:
```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

kubectl -> authentication -> authorisation -> admission controllers -> create pod

Admission Controllers:
- Mutating - That can mutate a request
- Validating - That can validate a request
Mutating usually runs before validating

There are webhooks to create our own Admission Controllers
- Mutating Admission Webhook
- Validating Admission Webhook

```bash
kubectl-convert -f fileName --output-version versionName
```

```bash
kustomize build folderName | kubectl apply -f -
kubectl apply -k folderName

helm pull repo/imageName --untar
helm install repo/imageName
helm repo search imageName
helm hub search imageName
```

