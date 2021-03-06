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

### Lecture 58 - Volumes and autoprovisioning
- Documentation at http://kubernetes.io/docs/user-guide/persistent-volumes/
- All the different types available here: https://kubernetes.io/docs/concepts/storage/storage-classes/#parameters
- kubernetes plugin can autoprovision storage
1. Object of kind `StorageClass`
- Specify `type`, `zone` and `provisioner`
2. Object of kind `PersistentVolumeClaim`
- Specifies size and such...

### [Incomplete] Lecture 59 - wordpress with volume
TODO: Translate all AWS commands to GCP.

- The GCP to AWS conversions were kind of hard...
-- StorageClass: `kubernetes.io/aws-ebs` -> `kubernetes.io/gce-pd`
-- Cmdline: Use `gcloud` instead of `aws`
-- NFS: `aws efs create-file-system` ->

Continue by going here:
https://console.cloud.google.com/networking/networks/add?project=market-navigator-281018
gcloud filestore instances create wordpressvolume   --description="kubernetes course testing wordpress volume" --file-share=name=WORDPRESS_VOUME,capacity=1TB   --network=name=us-central1 --zone=us-west1-a

### Lecture 60 - Pod presets Theory
- Inject information into pod at runtime (secrets, configmaps, volumes, env variables, etc...)
- For `EnvVariables` & `VolumeMounts`, `PodPresets` applies to all containers in pod
- Easy way to avoid having to copy-paste the same configs for all pods sharing configurations

### [Incomplete] Lecture 61 - Pod presets Demo
TODO: Get PodPresets working on minikube.

$ kubectl get pp
$ kubectl create -f <pod-presets.yaml>

When starting the minikube cluster, we need:
$ minikube start --extra-config=apiserver.enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,PodPreset

### Lecture 62 - StatefulSets Theory
- StatefulSets (originally called Pet Sets) are used to run stateful applications
- Allow for a stable pod name (instead of podname-<randomstring>)
- Stateful apps allow stable storage with volumes based on number: podname-<stable_number>
- Stateful pods are not terminated in a random order:
    - Scaling up: Pods restart in order: 0 to n-1
    - Scaling down: Pods restart in reverse order: n-1 to 0

### Lecture 63 - StatefulSets Demo
- StatefulSets can be used for stateful apps (i.e. Cassandra) where an env variable
  needs to point to a very specific pod name.

### Lecture 64 - Daemon sets
- Guarantee exactly one pod of some type on every kubernetes node
- Use cases:
-- Logging aggreagator
-- Monitoring
-- Load balancers

### Lecture 65 - Resource usage monitoring
- "Heapster" enables monitoring and expose cluster metrics via REST endpoint
- Need a node with 3 pods: Heapster pod, InfluxDB pod, Grafan Pod

### Lecture 68 - Autoscaling
- Kubernetes supports autoscaling for `Deployments`, `ReplicationController`, or `ReplicaSet`
- Must start cluster with configurations `ENABLE_CUSTOM_METRICS=true`
- `horizontal-pod-austoscaler-sync-period` (defaults to checking heapster every 30 seconds)
- Update the `container` spec with the `resources: requests: cpu: 200m` config
- Must specify min/max replica counts
- Must specify target CPU/memory utilization

### Lecture 69 - Autoscaling Demo
- Tip for testing: create a bash loop running a bunch of `wget` requests until
  actual usage exceeds target usage.

### Lecture 70 - Affinity Theory
- Create rules that are `preferences` rather than `requirments`.
-- Specify a node label, but it would only be a nice-to-have.
- Available options: Node/Pod Affinity/anti-affinity
- Specify `weight` for `preferred rules` to change importance. Higher score -> precedence.

### Lecture 71 - Affinity demo
- Get nodes: `kubectl get nodes`
- Label a node: `kubectl label node <node-name> key=value`

### Lecture 72 - Interpod Affinity Theory
- Useful for co-located pods (i.e. two pods are always on the same node).
-- Use case: redis cache pod is in the same node as server pod
- If `podAffinity` filters match, pods are scheduled on nodes with matching `topologyKey`
- Pod `anti-affinity` is also an option

### Lecture 75: Taints and Tolerations
- Opposite of affinity to avoid specific pods/nodes
- `Tolerations`: Label applied on pod
- `Taints`: Label applied on a node
- Example command: `kubectl taint nodes <node_name> key=value:NoSchedule`
- Taints do not apply to pods already running except for `NoExecute`.
- Lots of options to tain by hardware, location, user, etc...

### Lecture 79: Postgres demo
- From lecture: https://github.com/CrunchyData/postgres-operator
- Another good reference: https://github.com/GoogleCloudPlatform/gke-cloud-sql-postgres-demo
- Kind of a boring lecture but following the READMe should be enough....

### Lecture 80 - Kubernetes Master service
- `kubectl` (authorized) -> Kube API server (REST service on master components)
- Other master components:
-- Scheduler
-- Controllers (replica controller, deployment, etc...)
- Non-master components:
-- Each node has a `kublet`
-- Each node has many pods
-- Each pod has many controllers
-- `kublet` is triggered by the master kube API server
- Other services:
-- etcd is a distributed key-value data store (3-5 nodes)

### Lecture 81 - Resource Quotas
- `request capacity` and `capacity limits`
- Recall: 200m = 200 millicpu (200 millicores); 205 of a CPU on the running node.
- Admin can set defaults for a specific namespace.
-- `requests.(cpu|mem|storage)` is max for all pods together
-- `limits.(cpu|memory)` is max for all pods together
- Admin can set max on deployment object liits (pods, configmaps, etc...)

### Lecture 82 - namespaces
- Virtual clusters; logical separation of clusters.
- Intended for multiple teams/projects; don't need for a single person.
- Quotas can be specified per namespace.
- Note: consider restricting loadblancers because they cost a lot of money.

### Lecture 84 - User Management Theory
- `Normal User`
-- Used to access the user externally (e.g. kubectl)
-- Not managed using objects
-- Access is limited through authorization
- `Service User`
-- Authenticate within the cluster (e.g. inside pod or kubelete)
-- Managed like secrets

### Lecture 85 - User Management Demo
1. `$ minikube ssh`
2. Create key using `openssl req` command
3. Create certificate using `openssl x509` command
5. Exit ssh session.
4. Copy key & certificate to ~/.minikube/<name>{.key,.crt}.
6. Append new configs to ~/.kube/config (see more detailed instructions).

### Lecture 86 - RBAC theory
- `Node` auth module: requests made by `kubeletes`
- `ABAC` auth module: policies & attributes.
- `RBAC` auth module: role based access control (dynamic config policities).
-- `authorization-mode` = RBAC.
- `Webhooks` auth module:
-`minikube`: $minikube start --extra-config=apiserver.Authorization.Mode=RBAC
- namespacing: `Role` for a single namespace or `ClusterRole` for all namespaces

### Lecture 87 - RBAC Demo
- Instructions to create a new user are at users/README.md.
- Recall contexts:
-- $ kubectl config get-contexts
-- $ kubectl config set-context <context-name> --cluster=<cluster_name> --user <user_name>
- `Role` defines the types of `verbs` one can do on different `resources`
- `RoleBinding` links a `Role` to a specific `user`.

### Lecture 88 - Networking
- `container-to-container`: within a pod through localhost + port number
- `pod-to-service`: DNS + NodePort
- `external-to-service`: LoadBalancer + NodePort
- `pod-to-pod`: communicate via pod IP addresses.
- Single cluster can't have more than 50 notes.
- Many different alternatives/options to handle networking (e.g. Flannel)

### Lecture 89 - Node Maintenance
- `NodeController` does the followings:
-- Keeps list of NodeSelector
-- Assigns IP space
-- Monitors health of nodes and handles rescheduling if necessary
- Adding new node
-- `kubelet` attempts self registration
-- Adds labels and metadata to the new node.
- Graceful node drainage: $ kubectl  drain nodename --frace-period=600
- Forced node drainage: $ kubectl  drain nodename --force

### Lecture 90 - High Availability (HA)
- etcd (distributed key-value store) used to manager multiple master NodeSelector
  by the load balancer.
- minikube is a 1 node cluster (no need for HA)

### Lecture 91 - TLS on ELB
- Cloud specific features (e.g. TLS termination in AWS ) can be configured with kube
- Use Kubernetes `Annotation` - many different types.

### Lecture 95 - Admission Controllers
- A lot of different pro/extensive features:
-- Object mutation.
-- Webhooks.
-- Validation
-- Node restrictions.
-- Node defaults.

### Lecture 96 - Pod security theory
- Enable/disable root mode.
- Specify UID/GUID for processes.
- `PodSecurityPolicies`
-- System process
-- User process

### Lecture 97 - Pod security Demo
- Need to enable `--extra-config=PodSecurityPolicy` when starting minikube
- In the deployment spec, specify `kubeAPIServer` of kind `Cluster`
- Object of kind `PodSecurityPolicy` that must be `privileged`
-- spec must define all privelege related documents
-- specify `rules` in an object of type `ClusterRole`
-- `ClusterRole` are bound using objects of type `RoleBindging`
- If `Dockerfile` contains `USER app`
-- Kubernetes cannot check the `UID` of `app`. Even if `app` was a privelerged,
   user, it would still error.

### Lecture 98 - etcd
- Distributed
- Reliable
- Fast (10k writes / second)
- Consensus algorithm: RAFT
- Heartbit timeout needs to be managed if etcd spans multiple data centers
- etcd requires quoram on writes

### Lecture 98 - Raft Consensus Algorithm
- Similar fault-taulrence as `Paxos`
- Amazing visualization: https://raft.github.io/
- Elect a leader and all the followers
