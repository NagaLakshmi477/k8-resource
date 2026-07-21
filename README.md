node js java ---> projects ---> we need libraries
what is image ---> base os + system packages + application run time + application dependies/libaries
but
in python ---> we have requrements.txt ---> running like a sysytem packages


what is docker architecture:
=================================
we have 3 components in docker
1.build the images
2.run/configure the image

client ---> Doker cli --> docker images and docker run ....
docker host----> where the docker is runnning , docker deamon( continously running)
repo ----> local repo and centarl repo
doker run nginx:
1. first it will check whether the image in local or not
2. if it is exstis then it will create the container
3. if not we will take/pull from the hub and create the container and sends output to client
4. if there are docker volumes and networking we can configure

disadvanatgs of docker:
==========================
auto scaling: no default auto scaling
load blancer : no load blance compomemts to blance the traffic b/w multiple containers
reliablity : if containers crashes, it will not restart by it's own(self healing)
what if docker host crash ? what about the storage ? 
if docker host crash we will loose the data (beacause dokcer is managing all volumes  at the same host)
networking : networking  is only bridge mode, if you have multiple bridge hosts bridge network will not work(3 instnces) ---> docker install(frontend,backend,db) ---> connection is very diffuclut

container orchestartor:
=======================
what is the docker orchestartor ?
docker swarm
---------------
1. this is paid software
It conatin multiple docker host it manges (networking).

Kubernetes:
============
autoscaling is easy, storage is outside of the cluster, rollbacks and multiple deployement strategies are available in k8's.
networking between containers is easy

what is cluster?(k8)
================
one master and one or multiple nodes
one master we can't run workloads on control plain
the conatiners are running on the nodes and control plain is cooridinates b/w them
if the one node is crash then it shift the containers to remaing node

what about data??
we are not save the data in cluster we will save ouside the cluster like volumes
docker files --> git (upload)--->  server ---> installing docker ---> pull from the git build as a image---> again pushing to ----> docker hub ---> image is ready

Developer
    |
    | eksctl create cluster
    v
AWS EKS
    |
    +--> Creates Control Plane
    |
    +--> Creates Worker Node 1 (EC2)
    |
    +--> Creates Worker Node 2 (EC2)
    |
    +--> Creates Worker Node 3 (EC2)

This entire setup = Kubernetes Cluster

how to run created image?
we will use manfest files ---> pushing into git ---> 

master (Amazon EKS)
nodes(Ec2 instances)
how to craete cluster?
we can use terraform to create clusters
we have ekctl(CMD) based on this we can use to create clusters.
After creating cluster we need to intract with by using cubectl
for authonitication purpose we will configure aws cli

workstation:
===============
1.install docker
2.run aws configure
3.install ekctl for cluster creation
4.install kubectl for cluster intraction


ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
eksctl version

# Install kubectl for cluster interaction

curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.0/2025-05-01/bin/linux/amd64/kubectl

# Make kubectl executable
chmod +x ./kubectl

# Move kubectl to a directory in PATH
sudo mv kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client

command:
======
aws configure 
installing ekscluster ---eks cluster install
installing cubectl ---cubectl cluster install --> aws doc

cluster creation:
===============
before that we need to do the configuration like how many nodes we need ,which region

managed node group: 
no need to manage required software and package installation manuvally aws manage everything here

spot instances:
================
on demand ---> whenever you require you will create an instance, aws gaurentees you get instance ---> 1$/hr
spot ---> AWS data centre have lot of resouces
i can give for less cost 70-80 per discount
2 min notification
on demadn ---> aws takes back the spot instances if it need capacity

so we can't use spot instances in prod level beacuse aws kills any time


commands:
===========
cd docker-file

eksctl  create cluster --config-file=eks.yaml
eksctl create nodegroup --config-file=eks.yaml
eksctl get nodegroup --cluster roboshop --region us-east-1
Now cluster is created we need to intract with nodes
kubectl get nodes --> shows the worker node

k8S is a ----> PAAS service
we can create multiple projects in single cluster
namespace ---> isolated project space in k8 where you can create your workloads
everything is yaml in k8

creating k8 name space:
apiversion:v1
kind: Namespace
metadata:
    name: developement
    labeles:
        project: roboshop
        environemnet: dev
kubectl apply -f <filename> --->> to apply
kubectl get namespaces ---> get the avilable namespaces
kubectl delete <namespace_id>--> to delete the namespace

pod is a resouce it is like a container/ smallest deployabl uint. It can contain one or multiple containers

kubectl apply -f <filename> --->> to apply
kubectl get nodes ---> to see the nodes
verything resource in a k8
kubectl delete -f <filename>
to delete cluster:
==========
eksctl delete cluster --config-file=eks.yaml


yaml slection:
---------------
apiVersion: v1
kind:
metadata:
    name:
    lables:
spec:
    containers:
    - name:
     image 

    - name:
     image:


------------------------------

NAME                   READY   STATUS             RESTARTS        AGE
multi-container-demo   1/2     CrashLoopBackOff   8 (2m11s ago)   18m
nginx                  1/1     Running            0               120m

---------------------------------
here multi-container-demo is not running properly
to see:
--------
watch kubectl get pods
CrashLoopBackOff   ----> error
to see the exact issue:
kubectl describe pod <podname>
Back-off restarting failed container almalinux in pod multi-container_defaut ---> output

here for nginx we have deamon to run server continously but for almalinux this is only os it doesn'thave any command for conatiner busy --no
so we can use sleep command

kubectl apply -f <filename>

it will show again an error like it will only take the updated changes that are in .image
so we can delete and craete agin
kubectl delete -f 03-multi-container.yaml   

what is the comjmand to login inside the conatiner:
----------------------------------------------------
kubectl exec -it nginx --bash 
kubectl exec -it multi-conatiner -c almalinux -- bash




to check the logs ---> kubectl describe pod <name>

container vs pod:
================
pod is the smallest deplaoyable unit in k8s. a pod can ahve one or more conatiner
all containers inside pod share same network and storage

why pod conatin 2 containers:
------------------------------
It took the same storage. It continously write the logs in the /var/logs location. logs are eferemal(temporay)
we will store the logs in elastic serach. It is a log storage tool

frontend ---> request ---> pod1 === It will take the request
Here we will run sidecar
pod2 ---> to store the logs.----> push ----> elastic search


multi containers are useful in sidecar patters and proxy patters. proxy means 1st proxy container gets the request, it checks whether the request shoild be forward to main containeror not

catalogue,user,cart,shipping,payment frontend can be in same pod as multiple contianers but we need to use diffrent ports opened by conatiners

labels:
========
lables are used as a selector for other resources usedful for filterring resources.
lables values can't keep long values only 63 char

annotations are similar to labels, it is key value pairs but not used as selectors,
annotations are used as a external resources for k8
we keep external information build url, image registry....
annotations values can be 256

resource limiting:
===================
continer main adavanatges ---> resources usage ---> dynamically
but if only one conainer will use all cpu and ram that why we need to limit the resources

horizonatal scaling vs vertical scaling:
========================================
1 house 20 years back ---> extra 4 members adeed ---> if we add one more floor on top of this
you should not stay while remodling the house
you basement may not strong to bear another floors

This is called vertical scaling, it is single point of falure and it conain downtime


a server is full  we need to stop and pod and increase the resource capacity. It have downtime and single point of failure.
here we can incraese the resources in server
Horizonatal scaling:
------------------------------
we can increase the number of serves
1 pod is busy then we can create another pod
It doesn't have downtime

kubectl top pods ----> to see the memory usage

kubectl exec -it env-var-demo -- bash
env

try to keep configuration outside --> loose coupling
we try keep configurations ouside the pod defination --> configMap means key value pair
we created podconfig we need to attach to pod

kubectl exec -it configmap-env -- bash

secrets:
echo -n "admin" | base64 --> It will some value
echo -n "admin321" | base64 --> it wil use for user namew and passwords

Service mesh:
===================
pod to pod communication, IP address is not useful since it is ephemeral.
we have services in k8's to achieve 
1. pod to pod communication
2. load blancer

Service:
==========
pod to pod commnication, Ip address is not useful since it is ephemeral. we have services in k8's
1.pod to pod communication
2.load blancer

ex: 
we are running 2 pods, so if we are running 2 pods means we need load blancer
for pod to pod communicationa and for load blancing we have service

to get pods--->kubectlget pods
tp get services ---> kubectl get services
to describe the service ----> kubectl describe svc nginx

service end point is ---> pod IP 
to see the pod Ip ---> kubectl get pods -o wide 

Here service name will work on DNS names
kubectl describe svc nginx
dns will work based on the service names

In service we have 3 types
-------------------------
1. cluster IP ----> Internal to the cluster means within the cluster
2. nodePort ---> It will open one port called node port in every node, external expose
3. Load blancer ----> It will create load blancer and nodeport in all nodes

Cluster Ip is created by deafult if we haven't mention any type
cluster Ip means this for internall service, means within the cluster

Now we haven't expose the application to outside

Now we will expose to outside

If anyone want to expose the application to outside we can use
nodeport and load blancer

nodeport:
========

EC2 == nodes

we need to allow the sg 
ec2 ---> sg group ---> sg_id ---> edit ---> custom tcp --->32752 ---> anywhere ---> 0.0.0.0/0 ---> done
then we can able to acces the application
we can call node port Ip is cluster Ip and it has some extra capabilites openminga port in worker node
kubectl apply -f 13-service-np.yaml
kubectl get svc
cluster IP is a subset of node port
extra capabilities:
---------
if any user can use the ip(3 ec2) we can able to login. It will open ports ina ll nodes
we need to allow the sg then it canable to acess

load blancer:
===============
load blancer contain cluster Ip, node port and one load blancer 
load blancer ---> node on Nodeport ----> cluster Ip  ----> pod

 reuest --> loadbalncer --> node on node port --> cluster Ip --> Pod
 serviec is not but dns

How i can create multiple pods to same image:
---------------------------------------------
nginx-c6h7j == replica-name-<random-5digits-code>

If any pod delete by mistakel the replaica will create automatically 
pod is a subset of replica set

Deployement:
===========
a version change is there 
v1 code ----> changes ----> v2
1. delete the v1 application
2. download the v2 application
3. restart

pod is the subset of replica set
replica is the subset of deplaoyement


install kubec
https://github.com/ahmetb/kubectx
k9s