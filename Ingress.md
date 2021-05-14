# Ingress

Brief explanation of the concept of `Ingress` or `Ingress Controller` in Kubernetes.

## The Problem

To export services from the cluster to the outside (public Internet) Kubernetes offers `services` with the type `NodePort` or `LoadBalancer`.
Both come with disadvantages which the `Ingress` concept tries to minimize.

### NodePort

A `NodePort` is a `service` which exposes a `Deployment` on a random (or user-suggested port) in the range of `30000` - `32767` on all Nodes. So your `service` would be reachable on any `Node` IP and the assigned port (the Kubernetes network layer would route traffic to the correct `Node` which hosts this `service`).

This port range (between `30000` - `32767`) does not cover the standard ports for HTTP services (`80`), HTTPS (`443`) or any other.

So this solution is quite unflexible and not standard conform.

### LoadBalancer

A `LoadBalancer` is a `service` which connects automatically to a LoadBalancer from your Cloud provider - this `service` is also only working _with_ Cloud providers and will create a new LoadBalancer per Kubernetes `LoadBalancer` `service` manifest.

This solution is thus quite expensive and requires a Cloud environment to run in.

## Ingress Controller

An `Ingress Controller` is a pod like any other pod in your cluster but service as a reverse proxy (HAProxy, Nginx) with layer 7 LoadBalancing capabilites.
Since the `Ingress Controller` is only a pod in the cluster it has the same restrictions (walled in) and no direct access from the outside - So an `Ingress Controller` would again use either a `NodePort` or a `LoadBalancer` to expose itself but this time it is a single point of entry for all app routings in the whole cluster.