# MicroService Design Patterns

Cheat sheet for the most used micro service patterns in the Kubernetes domain

![Summary of all patterns](https://matthewpalmer.net/kubernetes-app-developer/multi-container-pod-design.png)

## Sidecar Pattern

A sidecar service is not necessarily part of the primary application but connects to it and augments the functionality.

- Supporting service
- deployed together with the primary application
- each (primary) application has its own sidecar

![sidecar picture](https://docs.microsoft.com/en-us/azure/architecture/patterns/_images/sidecar.png)

An often used sidecar pattern usage is the extenstions of a logging mechanism to the primary application. I.e. the sidecar service would read a log file created and written to by the primary application and export those logs to a centralized logging service.

## Ambassador Pattern

An ambassador container resembles a special kind of [Sidecar](#sidecar-pattern) which handles requests and responses from the primary application (here the client) to an external service (outside this container).

- Proxy from primary app (client) to external service
- Handle security, connection retries, monitoring, statistics
- Prepare primary app (client) requests to the external service
- Prepare the response from the external service to the primary app (client)

![ambassador picture](https://docs.microsoft.com/en-us/azure/architecture/patterns/_images/ambassador.png)

For example some extenral services might be dynamic in nature or expect a specific format or return a format not suitable for the primary application. In such a case the ambassador is responsible for serving the primary application as an easy-to-use and predictable endpoint proxy.

## Adapter Pattern

An adapter is also a special kind of [Sidecar](#sidecar-pattern) which is used for converting the main application exposed API to a format an external client expects. So it is like a reversed [Ambassador](#ambassador-pattern).

It can also be used simply to export the primary application service port to a port the container is expected to export - and the primary application might have this hard-coded and thus an adapter might be the simplest solution.

- Export primary app on a different port
- Endpoint for the external service

![adapter picture](https://ansilh.com/08-multi_container_pod/04-pod-patterns/adapter.png?classes=shadow)

Mostly it is used when it is not feasible to change the existing code base of the primary application but you still want it to behave in a way your microservice architecture requires it to.