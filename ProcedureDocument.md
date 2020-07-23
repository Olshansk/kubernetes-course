Procedure Document
Kubernetes Procedure Document
Github repository [Read this first]
Download all the course material from: https://github.com/wardviaene/kubernetes-course

Kubernetes releases minor version updates of its distribution every 3 months

Rather than updating the scripts in the video lectures, the repository in Github is updated if any script need changes

The changes are often very minor, the API is very stable. Often API versions like v1betaX change to v1betaX+1 or to v1 (stable)

All the scripts you can find in the repository should work with the latest version of Kubernetes, if you have any issues, contact me through one of the channels listed below

Kubernetes setup lectures
There are multiple ways to setup a kubernetes cluster. You only need 1 working cluster to do the demos, but I've added different ways that you can create your cluster.

A local cluster (on your machine): you can follow the minikube or docker for windows/mac lectures

A production cluster using Kops on AWS

A on-prem or cloud-agnistic cluster using kubeadm (lecture is at the end of the course)

A managed production cluster on AWS using EKS (lecture can also be found at the end of the course)

If you want to test all features, kops is the best choice. If you want to have a local cluster, try out minikube first. If you have issues getting minikube up, docker for windows/mac comes with kubernetes and is a great alternative.

Kops gives you full access to the master nodes. To learn to work with Kubernetes, kops is preferred. If you have issues setting up kops, you can give EKS a try (lectures at the end of the course). EKS is more expensive than kops, so make sure to keep an eye on your billing and destroy the cluster after you're done with testing/demos.

If you want to deploy a cluster on any cloud or on an on-prem machine, then kubeadm will be the way to go. If you want to use a cloud provider like Azure, Google, or DigitalOcean, that will also work. They all provide their own managed kubernetes solutions.

Slides
The slides can be downloaded from: https://d3jb1lt6v0nddd.cloudfront.net/udemy/Learn+DevOps+-+Kubernetes.pdf

Questions?
Send me a message

Use Q&A

Join our facebook group: https://www.facebook.com/groups/840062592768074/

Download Kubectl
Linux: https://storage.googleapis.com/kubernetes-release/release/v1.17.0/bin/linux/amd64/kubectl

MacOS: https://storage.googleapis.com/kubernetes-release/release/v1.17.0/bin/darwin/amd64/kubectl

Windows: https://storage.googleapis.com/kubernetes-release/release/v1.17.0/bin/windows/amd64/kubectl.exe

Or use a packaged version for your OS: see https://kubernetes.io/docs/tasks/tools/install-kubectl/

Minikube
Project URL: https://github.com/kubernetes/minikube

Latest Release and download instructions: https://github.com/kubernetes/minikube/releases

VirtualBox: http://www.virtualbox.org

Minikube on windows:
Download the latest minikube-version.exe

Rename the file to minikube.exe and put it in C:\minikube

Open a cmd (search for the app cmd or powershell)

Run: cd C:\minikube and enter minikube start

Test your cluster commands
Make sure your cluster is running, you can check with minikube status.

If your cluster is not running, enter minikube start first.

Run the hello-minikube deployment:

kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
kubectl expose deployment hello-minikube --type=NodePort --port=8080
then run:

minikube service hello-minikube --url

<open a browser and go to that url>

Kops
Project URL
https://github.com/kubernetes/kops

Free DNS Service
Sign up at http://freedns.afraid.org/

Choose for subdomain hosting

Enter the AWS nameservers given to you in route53 as nameservers for the subdomain

http://www.dot.tk/ provides a free .tk domain name you can use and you can point it to the amazon AWS nameservers

Namecheap.com often has promotions for tld’s like .co for just a couple of bucks



Cluster Commands
kops create cluster --name=kubernetes.newtech.academy --state=s3://kops-state-b429b --zones=eu-west-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=kubernetes.newtech.academy

kops update cluster kubernetes.newtech.academy --yes --state=s3://kops-state-b429b

kops delete cluster --name kubernetes.newtech.academy --state=s3://kops-state-b429b

kops delete cluster --name kubernetes.newtech.academy --state=s3://kops-state-b429b --yes

Kubernetes from scratch
You can setup your cluster manually from scratch

If you’re planning to deploy on AWS / Google / Azure, use the tools that are fit for these platforms

If you have an unsupported cloud platform, and you still want Kubernetes, you can install it manually

CoreOS + Kubernetes: ###a href="https://coreos.com/kubernetes/docs/latest/getting-started.html">https://coreos.com/kubernetes/docs/latest/getting-started.html

Docker
You can download Docker Engine for:

Windows: https://docs.docker.com/engine/installation/windows/

MacOS: https://docs.docker.com/engine/installation/mac/

Linux: https://docs.docker.com/engine/installation/linux/

DevOps box
Virtualbox: http://www.virtualbox.org

Vagrant: http://www.vagrantup.com

DevOps box: https://github.com/wardviaene/devops-box

Launch commands (in terminal / cmd / powershell):

cd devops-box/

vagrant up

Launch commands for a plain ubuntu box:

mkdir ubuntu

vagrant init ubuntu/xenial64

vagrant up
