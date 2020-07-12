# Volumes: attaching disk storage to containers

A volume is created when the pod is started and is destroyed when the pod is deleted.


## Volume types
- emptyDir: A simple empty directory used for storing transient data.
- hostPath: Used for mounting directories from the worker node’s filesystem into the pod.
- gitRepo: A volume initialized by checking out the contents of a Git repository.
nfs: An NFS share mounted into the pod.
- gcePersistentDisk (Google Compute Engine Persistent Disk), awsElasticBlockStore (Amazon Web Services Elastic Block Store Volume), azureDisk (Microsoft Azure Disk Volume): Used for mounting cloud provider-specific storage.
- cinder, cephfs, iscsi, flocker, glusterfs, quobyte, rbd, flexVolume, vsphere: Volume, photonPersistentDisk, scaleIO—Used for mounting other types of network storage.
- configMap, secret, downwardAPI: Special types of volumes used to expose certain Kubernetes resources and cluster information to the pod.
- persistentVolumeClaim: A way to use a pre- or dynamically provisioned persistent storage. (We’ll talk about them in the last section of this chapter.)

### emptyDir volume
An emptyDir volume is especially useful for sharing files between containers running in the same pod.

### hostPath Volume
A hostPath volume points to a specific file or directory on the node’s filesystem. Pods running on the same node and using the same path in their hostPath volume see the same files. HostPath volume’s contents are not deleted when pod is torn down.

## Decoupling pods from the underlying storage technology

### PersistentVolume and PersistentVolumeClaim
When creating the PersistentVolume, the admin specifies its size and the access modes it supports. When a cluster user needs to use persistent storage in one of their pods, they first create a PersistentVolumeClaim manifest, specifying the minimum size and the access mode they require. The user then submits the PersistentVolumeClaim manifest to the Kubernetes API server, and Kubernetes finds the appropriate PersistentVolume and binds the volume to the claim. The PersistentVolumeClaim can then be used as one of the volumes inside a pod. Other users cannot use the same PersistentVolume until it has been released by deleting the bound PersistentVolumeClaim.

> PersistentVolumes don’t belong to any namespace. They’re cluster-level resources like nodes.

#### Creating PersistentVolumeClaim

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mongodb-pvc
    spec:
      resources:
        requests:
          storage: 1Gi
      accessModes:
      - ReadWriteOnce
      storageClassName: ""

As soon as you create the claim, Kubernetes finds the appropriate PersistentVolume and binds it to the claim.

The claim is shown as Bound to PersistentVolume mongodb-pv. Note the abbreviations used for the access modes:

- RWO—ReadWriteOnce—Only a single node can mount the volume for reading and writing.
- ROX—ReadOnlyMany—Multiple nodes can mount the volume for reading.
- RWX—ReadWriteMany—Multiple nodes can mount the volume for both reading and writing.

#### Using PersistentVolumeClaim in a pod

    apiVersion: v1
    kind: Pod
    metadata:
      name: mongodb
    spec:
      containers:
      - image: mongo
        name: mongodb
        volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
        ports:
        - containerPort: 27017
          protocol: TCP
      volumes:
      - name: mongodb-data
        persistentVolumeClaim:
          claimName: mongodb-pvc

## StorageClass resource
The StorageClass resource specifies which provisioner should be used for provisioning the PersistentVolume when a PersistentVolumeClaim requests this StorageClass.

#### StorageClass definition

    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: fast
    provisioner: kubernetes.io/gce-pd
    parameters:
      type: pd-ssd
      zone: europe-west1-b

#### Requesting StorageClass

      apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mongodb-pvc
    spec:
      storageClassName: fast
      resources:
        requests:
          storage: 100Mi
      accessModes:
        - ReadWriteOnce

The nice thing about StorageClasses is the fact that claims refer to them by name. The PVC definitions are therefore portable across different clusters, as long as the StorageClass names are the same across all of them.

The best way to attach persistent storage to a pod is to only create the PVC (with an explicitly specified storageClassName if necessary) and the pod (which refers to the PVC by name). Everything else is taken care of by the dynamic PersistentVolume provisioner.

https://livebook.manning.com/book/kubernetes-in-action/chapter-6/303
1. Cluster admin sets up a PersistentVolume provisioner (if one's not already deployed)
2. Admin creates one or more StorageClasses and marks one as the default (it may already exist)
3. User creates a PVC referencing one of the Serviceses (or none to use the default)
4. Kubernetes looks up the StorageClass and the provisioner referenced in it and asks the provisioner to provision a new PV based on the PVC's requested access mode and storage size and the parameters in the StorageClass
5. Provisioner provisions the actual storage, creates a PersistentVolume, and binds it to the PVC
6. User creates a pod with a volume referencing the PVC by name
