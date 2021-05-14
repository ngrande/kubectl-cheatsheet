# Cheat Sheet

Treat the following document as a quick-reference to get example `YAML` and `shell` commands for creating/executing different Kubernetes objects. For in-depth explanations click on the provided __Kubernetes Doc__ links.

---

When running `kubectl` commands you can always add the flags `--dry-run=client -o yaml` to only show the corresponding YAML notation.

---

- [Cheat Sheet](#cheat-sheet)
  - [Enable completion](#enable-completion)
  - [Verb-driven commands](#verb-driven-commands)
    - [Run](#run)
    - [Expose](#expose)
      - [Create a deployment](#create-a-deployment)
      - [Expose the deployment](#expose-the-deployment)
      - [Test with busybox](#test-with-busybox)
    - [Autoscale](#autoscale)
  - [Object creation](#object-creation)
    - [Pods](#pods)
      - [YAML output](#yaml-output)
    - [Deployment](#deployment)
    - [Service](#service)
    - [Namespace](#namespace)
  - [Edit objects](#edit-objects)
  - [Listings](#listings)
    - [List objects](#list-objects)
      - [as YAML](#as-yaml)
      - [Show labels](#show-labels)
      - [Filter by labels](#filter-by-labels)
    - [Labels](#labels)
      - [Edit labels](#edit-labels)
      - [Remove labels](#remove-labels)
  - [Interact with pods](#interact-with-pods)
  - [Deployments](#deployments)
    - [Create a deployment](#create-a-deployment-1)
    - [Update the deployment](#update-the-deployment)
    - [Rollout history](#rollout-history)
    - [Undo rollout](#undo-rollout)
      - [Set to specific revision](#set-to-specific-revision)
  - [Volumes](#volumes)
    - [Persistent Volume](#persistent-volume)
    - [Persistent Volume Claim](#persistent-volume-claim)
    - [Claim as Volumes](#claim-as-volumes)
  - [Environment Variables](#environment-variables)
  - [Secrets](#secrets)
    - [Using Secrets as a file](#using-secrets-as-a-file)
    - [Using Secrets as Environment Variables](#using-secrets-as-environment-variables)
  - [ConfigMap](#configmap)
    - [Using ConfigMap as Environment Variable](#using-configmap-as-environment-variable)
    - [Using ConfigMap as a file](#using-configmap-as-a-file)
  - [Job](#job)
  - [CronJob](#cronjob)
  - [ServiceAccounts](#serviceaccounts)
    - [Create a ServiceAccount](#create-a-serviceaccount)
    - [Use a ServiceAccount](#use-a-serviceaccount)
  - [StatefulSets](#statefulsets)
  - [DaemonSet](#daemonset)
  - [To Add](#to-add)

## Enable completion

`source <(kubectl completion zsh)` or for bash `source <(kubectl completion bash)`

## Verb-driven commands

### Run

`kubectl run --image nginx <name>`

Run a particular image on the cluster

### Expose

`kubectl expose deployment <deployment-name> --port <port> --target-port <port>`

Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service

<details>
<summary>Example</summary>

#### Create a deployment

`kubectl create deployment nginx --image nginx --port 80`

#### Expose the deployment

on port 8080

`kubectl expose deployment nginx --port 8080 --target-port 80`

#### Test with busybox

`kubectl run --image busybox busybox --command sleep --command 300`

and `wget` it

`kubectl exec busybox -- wget nginx:8080`

</details>

### Autoscale

`kubectl autoscale deployment --max=<max> <name>`

Auto-scale a Deployment, ReplicaSet, StatefulSet, or ReplicationController

## Object creation

### Pods

Create and run a pod

without anything

`kubectl run --image nginx <name>`

setting a command

`kubectl run --image busybox <name> --command "sleep" --command "3600"`

setting ports

`kubectl run --image nginx <name> --port 80`

#### YAML output

`kubectl run --image nginx <name> --port 80 -o yaml --dry-run=client`

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### Deployment

Create a deployment

`kubectl create deployment --image=nginx --port 80 <name>`

### Service

Create a service

`kubectl create service clusterip --tcp 80 <name>`

### Namespace

Create a namespace

`kubectl create namespace <namespace-name>`

## Edit objects

It is also possible to edit objects like pods, deployments, services, etc. - but not all fields can be edited while the object is still active.
So sometimes it is required to recreate the object to make those changes.

`kubectl edit pod nginx`

## Listings

### List objects

`kubectl get pods`

```shell
  $ kubectl get pods
  NAME                     READY   STATUS        RESTARTS   AGE
  alpine                   1/1     Running       1          6m15s
  busybox                  1/1     Running       1          95m
  debian                   0/1     Terminating   0          2m52s
```

`kubectl get <deployments|namespaces|services|pods|...>`

#### as YAML

`kubectl get pod nginx -o yaml`

or better

`kubectl run --image nginx nginx --dry-run=client -o yaml`

#### Show labels

`kubectl get pods --show-labels`

#### Filter by labels

`kubectl get pods -l app=nginx`

`kubectl get pods -l 'app in (nginx, redis)'`

### Labels

#### Edit labels

`kubectl label pod nginx release=nginx-123`

#### Remove labels

`kubectl label pod nginx release-`

## Interact with pods

open a shell in a pod

`kubectl exec -it busybox -- sh`

or directly run a command

`kubectl exec busybox -- wget nginx`

## Deployments

[Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

A deployment is an object on top of a `ReplicaSet` which includes such a `ReplicaSet` of `Pods` and also manages rollouts of new updates.

### Create a deployment

`kubectl create deployment nginx --image nginx --port 80 --replicas 5`

### Update the deployment

To update, simply edit the YAML or the object itself

`kubectl edit deployment nginx`

and the `Deployment` object will handle the update / `rollout` (green/blue update, so not all replicas will be updated at the same time but rather _n_ at a time)

### Rollout history

`kubectl rollout history deployment nginx`

### Undo rollout

`kubectl rollout undo deployment nginx`

will set it back to one revision before the current

#### Set to specific revision

`kubectl rollout undo deployment nginx --to-revision=2`

## Volumes

[Kubernetes Docs](https://kubernetes.io/docs/concepts/storage/)

> Kubernetes supports many types of volumes. A Pod can use any number of volume types simultaneously. Ephemeral volume types have a lifetime of a pod, but persistent volumes exist beyond the lifetime of a pod. When a pod ceases to exist, Kubernetes destroys ephemeral volumes; however, Kubernetes does not destroy persistent volumes. For any kind of volume in a given pod, data is preserved across container restarts.

### Persistent Volume

[Kubernetes Docs](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)

Declare a `PersistentVolume` which provides several types of volumes (`AzureDisk`, `AzureFile`, `hostPath`, and more ...).
This Volume is persistent and available in the whole cluster. Pods can access this storage (full or only use a part of it) by using a [PersistentVolumeClaim](#persistent-volume-claim)

```YAML
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  storageClassName: local-storage
  capacity:
    storage: 2Gi 
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

### Persistent Volume Claim

[Kubernetes Docs](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

In order for a `Pod` to actually use a `PersistentVolume` it first must be bound to a `PersistentVolumeClaim` - later this claim is referred to in the `Pod`.

```YAML
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

### Claim as Volumes

[Kubernetes Docs](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
  - name: myapp
    image: busybox
    command: ["sleep", "300"]
    volumeMounts:
      - mountPath: "/tmp/mnt"
        name: my-volume
  volumes:
  - name: my-volume
    persistentVolumeClaim:
      claimName: my-pvc
```

## Environment Variables

[Kubernetes Docs](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/#define-an-environment-variable-for-a-container)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: pod-env-var
  labels:
    release: env
spec:
  containers:
  - name: env-var
    image: busybox
    command: ["sleep", "300"]
    env:
    - name: ENV_VAR_A
      value: "Hello from the environment"
    - name: ENV_VAR_B
      value: "Such a sweet sorrow"
```

## Secrets

[Kubernetes Doc](https://kubernetes.io/docs/concepts/configuration/secret/)

`data` secrets have to be `base64` encoded like

```shell
$ echo -n "this is my key" | base64
dGhpcyBpcyBteSBrZXk=
```

```YAML
apiVersion: v1
kind: Secret
metadata:
  name: secret-dockercfg
data:
  my-key: "dGhpcyBpcyBteSBrZXk="
```

or as simple strings with `stringData`

```YAML
apiVersion: v1
kind: Secret
metadata:
  name: secret-basic-auth
stringData:
  username: admin
  password: t0p-Secret
```

### Using Secrets as a file

[Kubernetes Docs](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: secret-basic-auth
```

### Using Secrets as Environment Variables

[Kubernetes Docs](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: secret-basic-auth
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: secret-basic-auth
            key: password
```

## ConfigMap

[Kubernetes Docs](https://kubernetes.io/docs/concepts/configuration/configmap/)

```YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-demo
data:
  # property-like keys; each key maps to a simple value
  number_of_players: "3"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5  
```

### Using ConfigMap as Environment Variable

[Kubernetes Docs](https://kubernetes.io/docs/concepts/configuration/configmap/)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod-env
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        - name: NUM_PLAYERS
          valueFrom:
            configMapKeyRef:
              name: configmap-demo
              key: number_of_players
```

### Using ConfigMap as a file

[Kubernetes Docs](https://kubernetes.io/docs/concepts/configuration/configmap/)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod-volume
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
    - name: config
      configMap:
        name: configmap-demo
        items:
        - key: "game.properties"
          path: "game.properties"
```

## Job

[Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)

> A Job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created. Suspending a Job will delete its active Pods until the Job is resumed again.

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

## CronJob

[Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)

> One CronJob object is like one line of a crontab (cron table) file. It runs a [Job](#job) periodically on a given schedule, written in Cron format.

```YAML
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

## ServiceAccounts

[Kubernetes Docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

> A service account provides an identity for processes that run in a Pod.

### Create a ServiceAccount

```YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
```

### Use a ServiceAccount

Use the `ServiceAccount` in a `Pod`. Each `Pod` can use exactly one `ServiceAccount` (and _must_ be from the same `Namespace`).

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: build-bot
  containers:
  - name: nginx
    image: nginx
```

## StatefulSets

[Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

> Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.

Example with a `headless Service`

```YAML
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None # headless
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "local-storage"
      resources:
        requests:
          storage: 1Gi
```

## DaemonSet

[Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

> A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

```YAML
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## To Add

- Add Doc links to every section
- Ingress