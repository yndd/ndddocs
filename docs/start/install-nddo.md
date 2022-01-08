# Installation

NDDO is an application that runs within a kubernetes cluster. Through customer resources it enables a declarative management of networking. 

## Pre-requisites

install ndd-core 

## Install nddo registries

There are 5 registries right now

- org-registry
- topo-registry
- ipam-registry
- ni-registry
- as-registry

### org-registry

create a file nddr-package-org-registry.yaml with the following content

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Intent
metadata:
  name: nddr-org-registry
  namespace: ndd-system
spec:
  package: yndd/nddr-org-registry:latest
  packagePullPolicy: Always
```

```
kubectl apply -f nddr-package-org-registry.yaml
```

after this command a pod should be create like this

```
k get pods -n ndd-system -w
NAME                                              READY   STATUS    RESTARTS   AGE
...
nddr-org-registry-ac199de081fe-5bf5bc7f96-c8sph   2/2     Running   0          12s
```

### topo-registry

create a file nddr-package-topo-registry.yaml with the following content

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Intent
metadata:
  name: nddr-topo-registry
  namespace: ndd-system
spec:
  package: yndd/nddr-topo-registry:latest
  packagePullPolicy: Always
```

```
kubectl apply -f nddr-package-topo-registry.yaml
```

after this command a pod should be create like this

```
k get pods -n ndd-system -w
NAME                                              READY   STATUS    RESTARTS   AGE
...
nddr-topo-registry-83d75d175ab9-fbcd55666-k9jtg   2/2     Running             0          7s
```

### ni-registry

create a file nddr-package-ni-registry.yaml with the following content

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Intent
metadata:
  name: nddr-ni-registry
  namespace: ndd-system
spec:
  package: yndd/nddr-ni-registry:latest
  packagePullPolicy: Always
```

```
kubectl apply -f nddr-package-ni-registry.yaml
```

after this command a pod should be create like this

```
k get pods -n ndd-system -w
NAME                                              READY   STATUS    RESTARTS   AGE
...
nddr-ni-registry-4a71a2c372d0-74589448f6-nw8hl    2/2     Running             0          7s
```

### as-registry

create a file nddr-package-as-registry.yaml with the following content

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Intent
metadata:
  name: nddr-as-registry
  namespace: ndd-system
spec:
  package: yndd/nddr-as-registry:latest
  packagePullPolicy: Always
```

```
kubectl apply -f nddr-package-as-registry.yaml
```

after this command a pod should be create like this

```
k get pods -n ndd-system -w
NAME                                              READY   STATUS    RESTARTS   AGE
...
nddr-as-registry-5d7fa458ac98-544588b5cf-66w2z    2/2     Running             0          6s
```

### ipam-registry

create a file nddr-package-ipam-registry.yaml with the following content

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Intent
metadata:
  name: nddr-ipam-registry
  namespace: ndd-system
spec:
  package: yndd/nddr-ipam-registry:latest
  packagePullPolicy: Always
```

```
kubectl apply -f nddr-package-ipam-registry.yaml
```

after this command a pod should be create like this

```
k get pods -n ndd-system -w
NAME                                              READY   STATUS    RESTARTS   AGE
...
nddr-as-registry-5d7fa458ac98-544588b5cf-66w2z    2/2     Running             0          6s
```

## Install nddo adaptors

install the adaptation layer which is used by the Intents to create the abstract data structures

### ndda-network

create a file ndda-package-network.yaml with the following content

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Intent
metadata:
  name: ndda-network
  namespace: ndd-system
spec:
  package: yndd/ndda-adaptor-network:latest
  packagePullPolicy: Always
```

```
kubectl apply -f ndda-package-network.yaml
```

after this command a pod should be create like this

```
k get pods -n ndd-system -w
NAME                                              READY   STATUS    RESTARTS   AGE
...
ndda-network-0f7367906ec0-6f84898f65-8f8c8         2/2     Running             0          6s
```

## Install nddo Intents

Install the intents, there are 2 right now:
- infrastructure
- Endpoint groups
- Vpc

### nddo-infrastructure

create a file nddo-package-infra.yaml with the following content

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Intent
metadata:
  name: nddo-infra
  namespace: ndd-system
spec:
  package: yndd/nddo-intent-infra:latest
  packagePullPolicy: Always
```

```
kubectl apply -f nddo-package-infra.yaml
```

after this command a pod should be create like this

```
k get pods -n ndd-system -w
NAME                                              READY   STATUS    RESTARTS   AGE
...
nddo-infra-0f86fd936790-5bbdbc9f85-l7tgp           2/2     Running             0          6s
```

### nddo-epgroup

create a file nddo-package-epgroup.yaml with the following content

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Intent
metadata:
  name: nddo-epgroup
  namespace: ndd-system
spec:
  package: yndd/nddo-intent-epgroup:latest
  packagePullPolicy: Always
```

```
kubectl apply -f nddo-package-epgroup.yaml
```

after this command a pod should be create like this

```
k get pods -n ndd-system -w
NAME                                              READY   STATUS    RESTARTS   AGE
...
nddo-infra-0f86fd936790-5bbdbc9f85-l7tgp           2/2     Running             0          6s
```