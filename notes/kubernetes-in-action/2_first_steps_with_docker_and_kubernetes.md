# First Steps with Docker and Kubernetes

#### Deploy app

    $ kubectl run kubia --image=luksa/kubia --port=8080 --generator=run/v1
    2 replicationcontroller "kubia" created

## Pod
A pod is a group of one or more tightly related containers that will always run together on the same worker node and inthe same Linux namespace(s). Each pod is like a separate logical machine with its own IP, hostname, processes, and so on, running a single application.

#### Listing pods

    $ kubectl get pods -o wide # -o option shows the pod's IP and the node the pod is running on

## Scheduling
Assigning the pod to a node. The pod is run immediately.

### Accessing application
#### Creating a Service object

    $ kubectl expose rc kubia --type=LoadBalancer --name kubia-http

#### Listing services

    $ kubectl get services
    $ kubectl get svc

## The logical parts of your system
The main and most important component in your system is the pod. It contains only a single container, but generally a pod can contain as many containers as you want. Inside the container is your Node.js process, which is bound to port 8080 and is waiting for HTTP requests. The pod has its own unique private IP address and hostname.


#### Horizontally scaling the application

    $ kubectl get replicationcontrollers
    $ kubectl get rc
    $ kubectl scale rc kubia --replicas=3

