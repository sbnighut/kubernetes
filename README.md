# kubernetes
This repository contains projects along with the kubernetes configuration for running a cluster in a production ready environment




**Kubernetes Steps and comands:**

1. Install kubectl (command line for managing kubernetes pods or cluster)
2. Install minikube (local installation for running kubernetes on local machine)
3. Minikube docker-env (lists the environment variables we need to set if we want to point to the docker installation inside the minikube instead of pointing to the MacOS docker installation. Run the eval command as a shortcut)
4. docker pull “docker hub image url along with the release tag if necessary”
5. docker image ls (to list all the downloaded images)
6. docker container run -p 83:80 -d 25be (runs the specific image with this prefix, and exposes the port 83 to outside world and redirects the traffic on 83 port number to port 80 internally, which is the nginx server runs on)
7. docker container stop 25be to stop the container
8. Run “minikube ip” (Note: since the we are running the image inside minikube’s docker installation we need to first grab minikube’s IP address and then hit the url. Final url would look something like http://192.168.99.101:83/)

 **Commands for creating a pod**
1. code pod-config.yaml (create the pod config file) 
2. Specify the pod name, and containers
3. Each container should have container name, and image location

Sample pod-config.yaml:
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: dummyapp
    image: richardchesterwood/k8s-fleetman-webapp-angular:release0
```

**Kubectl Commands for deploying pod and checking statuses**
1. kubectl get all (gets all the clusters currently running)
2. kubectl apply -f first-pod.yaml (use the config file to deploy the pod)
3. kubectl describe pod pod_name (describes all the status and info about the deployed pod)
4. kubectl -it exec pod_name sh  (Note: We cannot visit the pod through browser, but we can connect to the pod. The mentioned command executes the “ls” command on the pod’s container. We can infer the shell inside pod’s container and then trigger that shell command from browser. “It” means Get interactive with teletype emulation)
5. wget http://localhost:80 (We are now inside the container’s shell. We can execute any command in that shell. The mentioned command does an http get on the local url and saves that index.html file in the directory, which is root by default)

**Kubernetes Service: (This service is exposed by the cluster to the outside world through nodePort 30080. We will specify selector rules for selecting pods, ports configuration for routing and type of the service i.e. NodePort)**

*Our kubernete webapp-service configuration would look like this:*

```
spec:
  selector:
    # It defines which pods are going to be selected or considered
    # by this service
    # Select all the pods with value "webapp" for the label "mylabelname"
    mylabelname: webapp
  ports:
    - name: http
      port: 80 # This service is going to talk to port 80 of pods
      nodePort: 30080 #node port has a constraint to be >30000
      
    # type: LoadBalancer for load balancer
    # type: ClusterIP # if the service is only internally accessible
  type: NodePort # this means we are going to expose the port through the node, to the external world
  ```

  *Commands:*
1. kubectl apply -f webapp-service.yaml (this will create a kubernete service that will expose our internal *webapp pods* through port 30080, to the outside world)

  In order to access this pod from outside world we have to hit the url [minicube ip]:[service port] i.e. http://192.168.99.101:30080/. The selector rules will identify the pods and route to those pods.

**—————*————————————*——————————————*————————*————————**

**AWS Deployment related instruction:**
Create a bootstrap EC2 instance: This node is a physical server in the infrastructure
Master Node: Responsible for deciding which node should each pods be running on

1. Create an EC2 instance
2. Download the pem file containing private key
3. chmod 600 video-keypair (Change permission of this file to allow only rw for current user)
4. Run it on AWS
5. ssh -i video-keypair.pem ec2-user@3.17.140.224 (ssh to the EC2 server using)
6. Once you are inside the EC2 instance, install the kops tool for managing kubernetes cluster. Basically kops tool acts as “kubectl” for AWS

EC2 instance:
export NAME=sbnighut.k8s.local
export KOPS_STATE_STORE=s3://sbnighut-state-storage

kops create cluster --zones us-east-2a,us-east-2b,us-east-2c --name ${NAME}
kops create secret sshpublickey admin -i ~/.ssh/id_rsa.pub --name ${NAME}
kops edit ig nodes --name ${NAME}
kops update cluster ${NAME} --yes
Kops validate cluster

Delete cluster:
kops delete cluster --name ${NAME} --yes

Launch the k8s cluster:
Creating , editing , updating, applying yaml files

Bootstrap instance: ssh commands: 
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
export NAME=myfirstcluster.example.com
export KOPS_STATE_STORE=s3://prefix-example-com-state-store
export NAME=sbnighut.apps.com
export KOPS_STATE_STORE=s3://sbnighut-state-storage
export NAME=sbnighut.k8s.local
export NAME=sbnighut.apps.com
export KOPS_STATE_STORE=s3://sbnighut-state-storage
export NAME=sbnighut.k8s.local
export NAME=sbnighut.k8s.local
export KOPS_STATE_STORE=s3://sbnighut-state-storage
export NAME=sbnighut.k8s.local
export KOPS_STATE_STORE=s3://sbnighut-state-storage

**—————*————————————*——————————————*————————*————————**

**Ignore: Vehicle Tracker app related setup instructions**
If we want to deploy a new container locally with updated release version of code now
1. Update pod’s yaml configuration. Insert a ‘’’ string for creating additional pods definitions.
2. Update the kubernetes service to start the new released version of container 
3. Apply this pod file to kubectl and we will have two pods running simultaneously (Note: label value has to be string)
4. kubectl get pods --show-labels -l release=0 (It displays all the pods with specified label)
5. kubectl describe service angular-webapp (it shows which pods are the service connected to)

Uptil now we were responsible for managing the lifecycle of pods. Here onwards we will make replica sets of pods. We will maintain extra set of configurations for managing these replicas.

Replicaste yaml file that describes pods in a pod definition. And defines how many pods at minimum we want to maintain all the time.

Rollback deployment:
kubectl rollout status deploy webapp  (webapp is the deployment name)
kubectl rollout status status deplopy webapp (displays all the revisions did so far with the deployments)
kubectl rollout undo deploy webapp --to-revision=2 (will roll back the deployment the deployment to the specific version)

Segreggate resources into namespaces
How service discovery works:
Default namespace is “default”
Kubectl get ns (fetches all the namespaces)
kubectl get pods -n “namespace_name” (lists all the pods running under that namespace) 

Every kubernete cluster has a coredns service running that resolves name (service names to be precise) to ip addresses inside the cluster
kubectl exec -it webapp-6cc9cc96db-5vw44 sh (ssh into a random pod)
Nslookup service_name (This will call the coredns service for resolving IP of service_name)

Now install mysql client on the webapp node
apk add mysql-client
mysql -h database -uroot -ppassword fleetman (verify that the connection can be made to the mysql db server running on the database pod)

Inspect pods:
First do describe 
kubectl logs pod_name

Default Credentials for doing ssh into Minikube: (username: docker, password: tucker)
