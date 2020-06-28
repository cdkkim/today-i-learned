# Replication and other controllers: deploying managed pods

## Liveness probe
- HTTP GET probe

    livenessProbe:
        httpGet:
            path: /
            port: 8080
        initialDelaySeconds: 15
    
- TCP socket probe
- Exec probe

#### To see why the previous container terminated

    $ kubectl logs mypod --previous

## ReplicationControllers
A ReplicationController is a Kubernetes resource that ensures its pods are always kept running. 

### components
- label selector: determines what pods are in the ReplicationController’s scope
- replica count: specifies the desired number of pods that should be running
- pod template: used when creating new pod replicas

#### Hotrizontally scaling pods

    $ kubectl scale rc kubia --replicas=10
    $ kubectl edit rc kubia

#### Delete 

    $ kubectl delete rc kubia --cascade=False # keep its pod running

## ReplicaSets
new generation of ReplicationController and replaces it completely (ReplicationControllers will eventually be deprecated). The main improvements of ReplicaSets over ReplicationControllers are their more expressive label selectors.

### matchExpressions property

    selector:
       matchExpressions:
         - key: app
           operator: In
           values:
             - kubia

### About the API version attribute
Specifies two things
- api group(which is apps in this case)
- actual API version(v1beta2)

#### Creating and examining a ReplicaSet

    $ kubectl get rs
    $ kubectl describe rs

## DaemonSet
When you want a pod to run on each and every node in the cluster (and each node needs to run exactly one instance of the pod).

## Job
A completable task, after its process terminates, it should not be restarted again. Jobs are useful for ad hoc tasks, where it’s crucial that the task finishes properly. Jobs are part of the batch API group and v1 API version.

#### Scaling jobs

    $ kubectl scale job multi-completion-batch-job --replicas 3

## CronJob
Job resources will be created from the CronJob resource at approximately the scheduled time. The Job then creates the pods.

    apiVersion: batch/v1beta1
    kind: CronJob
    metadata:
      name: batch-job-every-fifteen-minutes
    spec:
      schedule: "0,15,30,45 * * * *"
      jobTemplate:
        spec:
          template:
            metadata:
              labels:
                app: periodic-batch-job
            spec:
              restartPolicy: OnFailure
              containers:
              - name: main
                image: luksa/batch-job


