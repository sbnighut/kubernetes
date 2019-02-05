# kubernetes
This repository contains projects along with the kubernetes configuration for running a cluster in a production ready environment




Kubernetes Steps and comands:

1. Install kubectl (command line for managing kubernetes pods or cluster)
2. Install minikube (local installation for running kubernetes on local machine)
3. Minikube docker-env (lists the environment variables we need to set if we want to point to the docker installation inside the minikube instead of pointing to the MacOS docker installation. Run the eval command as a shortcut)
4. docker pull “docker hub image url along with the release tag if necessary”
5. docker image ls (to list all the downloaded images)
6. docker container run -p 83:80 -d 25be (runs the specific image with this prefix, and exposes the port 83 to outside world and redirects the traffic on 83 port number to port 80 internally, which is the nginx server runs on)
7. docker container stop 25be to stop the container
8. Run “minikube ip” (Note: since the we are running the image inside minikube’s docker installation we need to first grab minikube’s IP address and then hit the url. Final url would look something like http://192.168.99.101:83/)

 Commands for creating a pod
1. code pod-config.yaml (create the pod config file) 
2. Specify the pod name, and containers
3. Each container should have container name, and image location

Sample pod-config.yaml:
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec: 
  containers:
  - name: dummyapp
    image: richardchesterwood/k8s-fleetman-webapp-angular:release0

Kubectl Commands for deploying pod and checking statuses
1. kubectl get all (gets all the clusters currently running)
2. kubectl apply -f first-pod.yaml (use the config file to deploy the pod)
3. kubectl describe pod pod_name (describes all the status and info about the deployed pod)
4. kubectl -it exec pod_name sh  (Note: We cannot visit the pod through browser, but we can connect to the pod. The mentioned command executes the “ls” command on the pod’s container. We can infer the shell inside pod’s container and then trigger that shell command from browser. “It” means Get interactive with teletype emulation)
5. wget http://localhost:80 (We are now inside the container’s shell. We can execute any command in that shell. The mentioned command does an http get on the local url and saves that index.html file in the directory, which is root by default)


