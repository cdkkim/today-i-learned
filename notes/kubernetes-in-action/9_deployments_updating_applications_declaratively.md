# Deployments: updating applications declaratively

To perform the rolling update

    kubectl rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2 --v 6

## Updating a Deployment

### RollingUpdate Strategy
The default strategy that removes old pods one by one while adding new ones at the same time, keeping the application available throughout the whole process, and ensuring there’s no drop in its capacity to handle requests. The upper and lower limits for the number of pods above or below the desired replica count are configurable.

#### slowing down the rolling update

    kubectl patch deployment kubia -p '{"spec": {"minReadySeconds": 10}}'

> The kubectl patch command is useful for modifying a single property or a limited number of properties of a resource without having to edit its defi- nition in a text editor.

### Recreate Strategy
Deletes all the old pods at once and then creates new ones, similar to modifying a ReplicationController’s pod template and then deleting all the pods.

| Method 	| What it does 	|
|-	|-	|
| kubectl edit 	| Opens the object’s manifest in your default editor. After making changes, saving the file, and exiting the editor, the object is updated. $ kubectl edit deployment kubia 	|
| kubectl patch 	| Modifies individual properties of an object. $ kubectl patch deployment kubia -p '{"spec":{"template": {"spec": {"containers": [{"name":"nodejs", "image": "luksa/kubia:v2"}]}}}}' 	|
| kubectl apply 	| Modifies the object by applying property values from a full YAML or JSON file. If the object specified in the YAML/JSON doesn’t exist yet, it’s created. The file needs to contain the full definition of the resource $ kubectl apply -f kubia-deployment-v2.yaml 	|
| kubectl replace 	| Replaces the object with a new one from a YAML/JSON file. In contrast to the apply command, this command requires the object to exist; otherwise it prints an error. $ kubectl replace -f kubia-deployment-v2.yaml 	|
| kubectl set image 	| Changes the container image defined in a Pod, ReplicationController’s template, Deployment, DaemonSet, Job, or ReplicaSet. $ kubectl set image deployment kubia nodejs=luksa/kubia:v2 	|

### Deployment rollback

    $ kubectl rollout status deployment kubia
    $ kubectl rollout undo deployment kubia
    deployment "kubia" rolled back

#### Diplaying a Deployment's rollout history

    $ kubectl rollout history deployment kubia

> Without the --record command-line option when creating the Deployment, the CHANGE-CAUSE column in the revision history would be empty, making it much harder to figure out what’s behind each revision.

#### Rolling back to a specific Deployment revision

    $ kubectl rollout undo deployment kubia --to-revision=1

#### Pausing the rollout process

    $ kubectl set image deployment kubia nodejs=luksa/kubia:v4
    deployment "kubia" image updated

    $ kubectl rollout pause deployment kubia
    deployment "kubia" paused

#### Resuming the rollout

    $ kubectl rollout resume deployment kubia
    deployment "kubia" resumed

> If a Deployment is paused, the undo command won’t undo it until you resume the Deployment.

### Blocking rollouts of bad versions
The `minReadySeconds` property specifies how long a newly created pod should be ready before the pod is treated as availabl

    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: kubia
    spec:
      replicas: 3
      minReadySeconds: 10 # You’re keeping minReadySeconds set to 10.
      strategy:
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 0 # You’re keeping maxUnavailable set to 0 to make the deployment replace pods one by one
        type: RollingUpdate

### Configuring a deadline for the rollout
If you use the kubectl describe deployment command, you’ll see it display a ProgressDeadlineExceeded condition, as shown in the following listing. The time after which the Deployment is considered failed is configurable through the progressDeadlineSeconds property in the Deployment spec. Because the rollout will never continue, the only thing to do now is abort the rollout by undoing it:

    $ kubectl rollout undo deployment kubia
    deployment "kubia" rolled back

