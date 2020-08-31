# Personal notes I took while watching the lectures

A great cheatsheet for kubectl: https://kubernetes.io/docs/reference/kubectl/cheatsheet/

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

### Lecture 27 - Replication controller theory
- Stateless application are much easier to scale horizontally
- Kubernetes containers are stateless (cannot be saved locally)
-- All session management needs to be done outside the container (object storage, database, etc...)
- Volumes can be used to run stateful apps
-- These containers are often scaled vertically rather than horizontally
- ReplicationController u sed to replicate pods

### Lecture 28 - Replication controller demo
- Find everything via: `kubectl get <nodes,service,deployments,pods,replicationcontrollers>`
- Easily delete pods via `kubectl delete pod <pod_name>`
-- The ReplicationController will automatically rescale
- Easily scale up/down via `kubectl scale --replicas=5 -f <filename>`
- Easily scale up/down via `kubectl scale --replicas=5 -f <filename>`
- Deleteing a replication controller: `kubectl delete rc helloworld-controller`

### Lecture 29 - Deployment theory
- Replica Set is a Replication controller with a bunch of extra filters
- Deployments are substitute for replication controllers/sets
-- Define a state and the deployment object makes sure the state is maintained
-- Create/Update/RollingUpdate
- Command cheatsheet:
-- kubectl get Deployments
-- kubectl get rs
-- kubectl get pods
-- kubectl set image <deployment>
-- kubectl edit <deployment>
-- kubectl rollout status <deployment>
-- kubectl rollout history <deployment>
-- kubectl rollout undo <deployment>


### Lecture 30 - Deployment demo
The takeaway is that it is easy to do rollouts, undo rollouts and jump
back and forth between versions by only changing the image and doing/undoing
rollouts.

$ minikube service helloworld-deployment --url
$ curl -X get http://192.168.64.2:30988
$ curl http://192.168.64.2:30988
$ kubectl set image deployment/helloworld-deployment k8s-demo=wardviaene/k8s-demo:2
$ kubectl rollout status  deployment/helloworld-deployment
$ kubectl rollout history
$ kubectl rollout undo deployment/helloworld-deployment
$ kubectl rollout history deployment/helloworld-deployment
$ kubectl get pods --show-labels

### Lecture 31 - Service Theory
- Pods are only accessible via a service
- Creating a service === creating an endpoint for the pod
- ClusterIP: Virtual IP address. Reachable only from within the cluster.
- NodePort: Same port on each node. Reachable externally.
- LoadBalancer: LoadBalancer by cloud provider (e.g. ELB on AWS) routes traffic to nodeports.
- DNS names:
-- External name: only enabled if DNS add-on is defined
- Reminder to self:
-- One pod can hold many containers
-- There is one "spec" definition in a yaml file for each container within a pod
-- Having a one-to-one mapping between pods and containers is a common practice
   but it is not mandatory and there are lots of use cases that avoid this
- Service can only run between ports 30000-32767

### Lecture 32 - Service Demo
Pretty straight forward, but just here are the main commands:
- kubectl get services
- kubectl get svc
- kubectl describe svc helloworld-service

### Lecture 33 - Labels Theory
- Same as tags in AWS
- Simply a key/value pair
- Not unique
- Label selectors can be used to filter the object (e.g. pod)
- Nodes:
-- Can be tagged with a specific label
-- NodeSelector can be added to pod to run on a specfific node

### Lecture 34 - Labels Demo
$ kubectl get nodes --show-labels
$ kubectl describe pod <pod_name> to investigate what's going on
- Note that if deployment/pod is launched with a node selector, and label
   has not been set yet, the pod will not run
$ kubectl label nodes minikube <key>=<value>

### Lecture 35 - Healthcheck Theory
- A running pod does not mean a functioning service: need health checks
- Two options:
-- Periodic command execution in container
-- Periodic HTTP request to an endpoint expose by container
- livenessProbe: configuration for a container spec (in a yaml file) for health checks

### Lecture 36 - Healthcheck Demo
- After creating the deployment/pod with a livenessProbe, it can be inspected via:
$ kubectl describe pod <pod_name>
$ kubectl describe deployment <deployment_name>
- To adjust the settings of a live <type>, use:
$ kubectl edit <type> <type_name>

### Lecture 37 - Readiness probe theory
- Liveness checks if pod is up
- Readiness checks if pod can accept traffic
- Example: need to warm up cache, etc...

### Lecture 39 - Pod state theory
- We can view all the namespaces and then get the state
  of the pods based on the namespace:
$ kubectl get namespaces
$ kubectl get pods -n <namespace>
- PodConditions provides more detail on state of pod (not documenting here)
- Pod states:
$ kubectl describe pod <pod_name> -n <namespace>
-- Pending: accepted but not running. Image is downloading, no resources, etc....
-- Succeeded: all containers within pod have been terminated.
-- Failed: at least one container within pod returned failure code.
- Container state:
$ kubectl describe pod <pod_name> -n <namespace> -o yaml

### Lecture 40 - Pod lifecycle theory
- Init container -> start hook -> readiness + liveness probes -> stop hook

### Lecture 41 - Pod lifecycle demo
- Busybox docker image (https://hub.docker.com/_/busybox) is a tiny image (1-5 Mb)
  that provides replacements for GNU utilities.
- Common patterns to investigate what's going on.
-- Terminal 1:
  $ watch -n1 kubectl get pods
-- Terminal 2:
  $ kubectl create -f <yaml_file>
-- Terminal 3:
  $ kubectl exec -it <pod_name> -- tail /timing -f
- The status of the pod can be watched/followed

### Lecture 42 - Secrets theory
- Way to distribute credentials, keys, password, etc...
- 3 options: External image, environment variables or files (volumes, file, dotenv files, etc...)
- Create a secret:
$ kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
- Check secrets:
$ kubectl get secrets
- Creating secrets can be done using YAML files with `kind: Secret`
- Spec container can be updated to have an environment variable with `valueFrom` `secretKeyRef`
- `volumeMounts` is the other option for secrets

### Lecture 43 - Secrets Demo
1. kubectl create -f <secret.yml>
2. kubectl create -f <pod-with-secrets-mount.yml>
3. kubectl describe pod <pod_name>  # Check the volume mounts
4. kubectl exec -it <pod_name> bash  # Enter the pod
5. cat /etc/creds/<secret_name>  # View the secrets

### Lecture 44 - Wordpress demo
- Create a "Secrets" yaml file containing namespace
- For the container spec in a Pod/Deployment type, setup env variables like so:
name: WORDPRESS_DB_PASSWORD
valueFrom:
  secretKeyRef:
    name: wordpress-secrets
    key: db-password
name: WORDPRESS_DB_HOST
value: 127.0.0.1
- Recall:
-- Use `kubectl create -f <>.yaml` to create secrets/deployments/services
-- Use `kubectl describe <Kind> <Name>` to get information
-- Use `minikube service wordpress-service --url` to get host:port of local server

### Lecture 45 - Web UI
$ minikube dashboard
$ minikube dashboard --url
$ kubectl config view

## Section 3

### Lecture 48 - Service Discovery Theory
- DNS service is an "addon" launched automatically
- Addons are in /etc/lubernetes/addons directory on master nodes
- Used within a pod to find other services in the same clusters
- Containers within a pod do not need service discovery
- Namespaces are used to logically separate pods within a cluster

Finding IP of the same service with different qualified names:
$ host app2-service
$ host app2-service.default
$ host app2-service.default.svc.cluster.local
Check namespaces:
$ cat /etc/resolv.conf for configurations

- Simply busybox shell:
$ kubectl run -i --tty busybox --image=busybox --restart=Never -- sh

### Lecture 49 - Service Discovery Demo
- Standard: Create a DB pod, a service, secrets, a deployment, etc...
- Testing service discovery:
1. kubectl get services # Get the <service_name> of interest
2. Start busybox with active shell(see command above)
3. nslookup <service_name>
- Easy way to find info about the service

### Lecture 50 - Config Map theory
- ConfigMap are not secrets
- One use case: A set of key-value pairs
- Another use case: full configuration files
- ConfigMap can be a mounted using volumes (i.e. inject into container at some path)
- Create configmap:
$ kubectl create configmap app-config --from-file=filename

- Use configmap with volumes:
-- volumeMount: specify `name` & `mountPath`
-- volume description: specify `name` and `configMap`

- Use configmap with env variables:
-- Create an env variable with a name and `valueFrom`
-- valueFrom uses `configMapKeyRef`

- Bash reminder to echo multiple lines
$ cat <<EOF> filename.txt
line 1
line 2
EOF

### Lecture 51 - Config Map Demo
$ kubectl get configmaps
$ kubectl create configmap nginx-config --from-file=configmap/reverseproxy.conf
$ kubectl get configmap nginx-config -o yaml

Configuration in pod description:
- Specify `volumeMounts` in the `containers` section
- Specify `volumes` in the section below

Checking by going into container:
$ kubectl exec -it helloworld-nginx bash
Catting the file path at the mounted volume path:
$ cat /etc/nginx/conf.d/reverseproxy.conf

Overall: Config files allow injection of configuration files or configuration data.

### Lecture 52 - Ingress Controller Theory
- Allows inbound connections to cluster (note that apiVersion is in beta)
- New `kind` of object: `Ingress`
- Ingress is an alternative to:
-- External LoadBalancer
-- nodeports
- Ingress controller can be custom or use one of the default kubectl ones
- Ingress types of rule:
-- Specify pod depending on host
-- Specify pod depending on path

### Lecture 53 - Ingress Controller Demo
- Adding ingress to minikube:
$ minikube addons enable ingress
- Opens port 80 (http) and port 443 (https)
- Recall `$minikube ip` to get the ip address
$ curl $(minikube ip) -H 'Host: helloworld-v2.example.com'


### Lecture 54 - External DNS Theory
- Ingress controllers can be used to reduce cost of LoadBalancers
-- Common pattern: 1 load balancer for all external traffic and send it to an ingress controller
- HTTP rules (host & prefix) can be used by ingress controller to route traffic

External DNS:
- Automatically creates new DNS record for every hostname in an EXTERNAL DNS server
- External DNS providers supporteted; Google CloudDNS< CLoudFlare, DigitalOcean, etc...

Flow of request:
1. Pod containing External DNS reads ingress rules.
1.1 This pod talks to external DNS server to create a record.
2. Pod containing ingress controller reads ingress rules
2.1 These are saved for a later date to forward requests accordingly.
3. Request comes in from the internet and go to external DNS Server.
3.1 External DNS returns IP Address of the load balancer.
3.2 Internet request is forwarded to the load balanacer.
3.2 Load balancer forwards request to ingress controller.
3.3 Ingress controller takes care of the rest.
* NOTE: The external dns server pod was only necessary in the configuration of
  the first step but is not involved in ongoing network requests.

### Lecture 55 - External DNS Theory
- `kubectl apply` can be used instead of `kubectl create` if we may want to modify configs in yaml file later
-- If something already exists and you run apply, it doesn't error (as it would with create).

### Lecture 56 - Volumes Theory
- Volumes store data outside container
- Container stop -> data is lost; make apps stateless.
- Persistent volumes attach volumes to a container that exist after container stops.
- Google Cloud volume plugin: Google Disk

When creating a volume, need to use aws/gcp tool to create a volume and specify:
- Size of volume
- Availability zone
- Etc...
* Note the availability zone

Creating pod with volume:
- Define a container with `volumeMount` (`mountPath` + `name`)
- Specify volumes in the container(`name` and `volumeId`)

### Lecture 57
* Demo used AWS
* GCP: https://cloud.google.com/persistent-disk
-- It has codelabs for postgresSQL database
-- Codelabs for persistent disk data
