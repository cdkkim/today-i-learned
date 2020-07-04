# Services: enabling clients to discover and talk to pods
A resource you create to make a single, constant point of entry to a group of pods providing the same service. Clients can open connections to that IP and port, and those connections are then routed to one of the pods backing that service.

non-Kubernets world: sysadmin would configure each client app by specifying the exact IP address or hostname of the server providing the service in the client’s configuration files

In Kubernetes
- Pods are ephemeral: They may come and go at any time, whether it’s because a pod is removed from a node to make room for other pods, because someone scaled down the number of pods, or because a cluster node has failed.
- Kubernetes assigns an IP address to a pod after the pod has been scheduled to a node and before it’s started: Clients thus can’t know the IP address of the server pod up front
- Horizontal scaling means multiple pods may provide the same service—Each of those pods has its own IP address. Clients shouldn’t care how many pods are backing the service and what their IPs are. They shouldn’t have to keep a list of all the individual IPs of pods. Instead, all those pods should be accessible through a single IP address.

## Create a Service object

### kubectl expose
#### through a YAML descriptor

    apiVersion: v1
    kind: Service
    metadata:
      name: kubia
    spec:
      ports:
      - port: 80
        targetPort: 8080
      selector:
        app: kubia


#### Remotely executing commands in running containers
    $ kubectl exec kubia-7nog1 -- curl -s http://10.111.249.153
    You've hit kubia-gzwli

The double dash (--) in the command signals the end of command options for kubectl. Everything after the double dash is the command that should be executed inside the pod.

#### Configuring session affinity on the service
To make all requests made by a certain client to be redirected to the same pod every time, you can set the service’s sessionAffinity property to ClientIP (instead of None, which is the default), as shown in the following listing.

