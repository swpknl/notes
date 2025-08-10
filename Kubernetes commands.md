Kubectl commands:

``` bash
kubectl get componentstatuses #Get the component status of the K8s cluster
kubectl config set-context my-context --namespace=mynamespace #Creates a new context
kubectl config use-context my-context #Use the newly created context
kubectl get pods,services #Get multiple objects
kubectl get pods --no-headers #Get without no headers
kubectl get pods --no-headers | wc -l #Get line count of the ouput of get
kubectl explain pods #fields in the K8s object
kubectl get pods --watch #Continously monitor the resource
kubectl apply -f fileName.yaml #Create or alter a K8s object
kubectl apply -f fileName.yaml --dry-run=client #Dry run creation without sending to server
kubectl edit resourceName objName #Edit a K8s object in file
kubectl delete -f fileName.yaml #Delete a K8s object
kubectl label pods bar color=red #Add the red color as label to the bar pod
kubectl label pods bar color- #Remove the color label bar pod
kubectl logs podName #Get the logs of the pod. It will show the logs and exit
kubectl logs podName -c containerName -f # Get the logs from a specific container and then stream the logs continously
kubectl exec -it podName -- bash # Execute commands in a running container
kubectl attach -it podName # Attach to a running process if bash is not available
kubectl top nodes # Shows list of resources in use by either pods or nodes
kubectl cordon # Prevents future pods from running on that machine
kubectl drain # Removes any pods that are currently running on that machine
kubectl uncordon # Re-enables pods scheduling onto the node
kubectl help commandName # Get help on a command
```

Pods:

```bash
kubectl run podName --image=imageName # Create a pod imperatively
kubectl get pods # Get the status of the pod
kubectl delete pods/podName # Delete the pod
kubectl describe pods podName # Describe the pod in detail
```

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: kuard
spec:
	containers:
		- image: gcr.io/kuar-demo/kuard-amd64:blue
		  name: kuard
		  ports:
			  - containerPort: 8080
				name: http
				protocol: TCP
```
