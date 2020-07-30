# Personal notes I took while watching the lectures

## Section 2

### Lecture 5
- Virtual machines ned a hypervisor, while containers do not
- Virtual machines include the guest OS, while containers do not

### Lecture 13
- The course goes over kops with AWS
- Great resource for kops with GCP: https://blog.mayadata.io/openebs/provisioning-google-cloud-with-k8s-using-it-in-house-tool-kops
- kops needs to hold state somewhere
  - We'll be using google cloud storage bucket (equivalent of S3)

# NOTE:  Finish the blog above before moving forward.

### Lecture 16
- Recall that we can run `minikube_docker_daemon` to have docker daemon point to minikube's daemon (don't need local docker running)
- Recall that `kubectl config get-contexts` shows all the contexts for which kubectl is configured (i.e. minikube, gcp cluster, aws cluster, etc...)

## Section 3

### Lecture 26
- Each pod can hold multiple containers
- Containers within a single pod can communicate over localhost
- Different pods can communicate with each other over the network - requires service discovery
Flow of a request:
1. Load balancers are publicly available (via internet)
2. Load balancer sends traffic to kubernetes iptables
3. Kubernetes iptables forwards traffic to the appropriate node (could be current node)
4. Kubernetes node forwards traffic to the appropriate pod
4.1 There is one yaml file per pod
4.2 There is a spec for each container within a pod (a list inside the yaml file)

### Lecture 27
- Stateless application are much easier to scale horizontally
- Kubernetes containers are stateless (cannot be saved locally)
-- All session management needs to be done outside the container (object storage, database, etc...)
- Volumes can be used to run stateful apps
-- These containers are often scaled vertically rather than horizontally
- ReplicationController used to replicate pods

### Lecture 28
- Find everything via: `kubectl get <nodes,service,deployments,pods,replicationcontrollers>`
- Easily delete pods via `kubectl delete pod <pod_name>`
-- The ReplicationController will automatically rescale
- Easily scale up/down via `kubectl scale --replicas=5 -f <filename>`
- Easily scale up/down via `kubectl scale --replicas=5 -f <filename>`
- Deleteing a replication controller: `kubectl delete rc helloworld-controller`
