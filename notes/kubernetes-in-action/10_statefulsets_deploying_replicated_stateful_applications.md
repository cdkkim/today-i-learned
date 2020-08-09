# StatefulSets: deploying replicated stateful applications

A StatefulSet is specifically tailored to applications where instances of the application must be treated as non-fungible individuals, with each one having a stable name and state. Each pod created by a StatefulSet is assigned an ordinal index (zero-based), which is then used to derive the pod’s name and hostname, and to attach stable storage to the pod.

StatefulSet requires you to create a corresponding governing headless Service that’s used to provide the actual network identity to each pod. Through this Service, each pod gets its own DNS entry, so its peers and possibly other clients in the cluster can address the pod by its hostname. A StatefulSet must guarantee at-most-one semantics for stateful pod instances.

## SRV records
SRV records are used to point to hostnames and ports of servers providing a specific service. Kubernetes creates SRV records to point to the hostnames of the pods back- ing a headless service.

#### List SRV records
    $ kubectl run -it srvlookup --image=tutum/dnsutils --rm
    --restart=Never -- dig SRV kubia.default.svc.cluster.local



