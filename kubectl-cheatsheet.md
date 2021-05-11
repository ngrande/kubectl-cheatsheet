# Cheat Sheet

When running those commands you can always add the flags `--dry-run=client -o yaml` to only show the corresponding YAML notation.

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

## To Add

- PersistentVolume
- PersistentVolumeClaims
- Secrets
- Environment Variables
- ConfigMap
- Job
- CronJob
- Ingress
- namespace
- serviceaccount