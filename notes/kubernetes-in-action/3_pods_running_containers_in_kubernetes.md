# Pods: running containers in Kubernetes

- A pod contain one or more containers
- A pod never spans multiple worker nodes
- Every pod can access every other pod at the other pod's IP address
- Pods are logical hosts and behave much like physical hosts or VMs in the non-container world

Containers are desinged to run only a single process per container(unless the process itself spawns child processes). Because you’re not supposed to group multiple processes into a single container, it’s obvious you need another higher-level construct that will allow you to bind containers together and manage them as a single unit. This is the reasoning behind pods.

#### Deciding when to use multiple containers in a pod
- Do they need to be run together or can they run on different hosts?
- Do they represent a single whole or are they independent components?
- Must they be scaled together or individually?

## Pod spec
- *Metadata* includes the name, namespace, labels, and other information about the pod.
- *Spec* contains the actual description of the pod’s contents, such as the pod’s containers, volumes, and other data.
- *Status* contains the current information about the running pod, such as what condition the pod is in, the description and status of each container, and the pod’s internal IP and other basic info.


#### Using kubectl explain to discover possible API object fields

    $ kubectl explain pods
    $ kubectl explain pod.spec

#### Retrieving the whold definition of a running pod

    $ kubectl get po kubia-manual -o yaml
    $ kubectl get po kubia-manual -o json

#### Viewing application logs

    $ kubectl logs <container id> -c <container name>

#### Forwarding a local network port to a port in the pod

    $ kubectl port-forward kubia-manual 8888:8080
    $ curl localhost:8888


#### Show labels

    $ kubectl get po --show-labels
    $ kubectl get po -L creation_method,env

#### Change labels

    $ kubectl label po kubia-manual creation_method-manual
    $ kubectl label po kubia-manual-v2 env=debug --overwrite

#### Get pods by label

    $ kubectl get po -l creation_method=manual
    $ kubectl get po -l env
    $ kubectl get po -l '!env'

#### Label nodes

    $ kubectl label node gke-kubia-85f6-node-0rrx gpu=true
    $ kubectl get nodes -l gpu=true

#### Annotate pod

    $ kubectl annotate pod kubia-manual mycompany.com/somannotation="foo bar"

#### Namespace

    $ kubectl get ns
    $ kubectl get po --namespace(-n) kube-system
    $ kubectl create namespace custom-namespace
    $ kubectl create -f kubia-manual.yaml -n custom-namespace

#### Delete pod

    $ kubectl delete po kubia-gpu
    $ kubectl delete po -l creation_method=manual
    $ kubectl delete po -l rel=canary
    $ kubectl delete ns custom-namespace
