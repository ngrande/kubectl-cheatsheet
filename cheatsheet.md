# Cheat Sheet

When running those commands you can always add the flags `--dry-run=client -o yaml` to only show the corresponding YAML notation.

## Verb-driven commands

### Run

`kubectl run --image nginx <name>`

Run a particular image on the cluster

### Expose

`kubectl expose pod <pod-name> --port <port>`

Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service

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


#### Show labels

`kubectl get pods --show-labels`

### Filter by labels

`kubectl get pods -l app=nginx`

`kubectl get pods -l 'app in (nginx, redis)'`

## Interact with pods

open a shell in a pod

`kubectl exec -it busybox -- sh`
