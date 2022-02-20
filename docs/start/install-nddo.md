# Installation

NDDO is an application that runs within a kubernetes cluster. Through customer resources it enables a declarative management of networking. 

## Pre-requisites

install ndd-core 

## Install nddo registries

There are 5 registries right now

- org-registry
- topo-registry
- ni-registry

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
nddo-epgroup-ebcddb8968e5-77756f9fbb-jrnl4         2/2     Running             0          6s
```

### nddo-vpc

create a file nddo-package-vpc.yaml with the following content

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Intent
metadata:
  name: nddo-vpc
  namespace: ndd-system
spec:
  package: yndd/nddo-intent-vpc:latest
  packagePullPolicy: Always
```

```
kubectl apply -f nddo-package-vpc.yaml
```

after this command a pod should be create like this

```
k get pods -n ndd-system -w
NAME                                              READY   STATUS    RESTARTS   AGE
...
nddo-vpc-8b8d4ae0e44c-7d4bd6b589-jg49s             2/2     Running             0          6s
```