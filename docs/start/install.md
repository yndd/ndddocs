# Installation

NDD is an application that runs within a kubernetes cluster. Through customer resources it enables a declarative management of network devices. 

## Pre-requisites

### Install a Kubernetest Cluster

Install a kubernetes cluster based on your preferences. Some examples are provided here for reference, but if you would use another kubernetes cluster flavor you can skip this step.

=== "kind cluster"

    Install the kind sw

    ```markdown
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.10.0/kind-linux-amd64
    chmod +x ./kind
    mv ./kind /some-dir-in-your-PATH/kind
    ```
    
    Create the kind cluster

    ```markdown
    kind create cluster --name ndd
    ```
    
    Check if the cluster is running

    ```markdown
    kubectl get node
    ```
   
    Expected output or similar

    ```markdown
    NAME                STATUS   ROLES    AGE     VERSION
    ndd-control-plane   Ready    master   2m58s   v1.19.1
    ```
    
    The full documentation for installing a kind can be found here: [kind installation](https://kind.sigs.k8s.io/docs/user/quick-start/)

=== "minikube cluster"

    ```markdown
    minikube getting started can be found here:  https://minikube.sigs.k8s.io/docs/start/
    ```

### Install a network device

Install a network device based on your preference. You can use container based labs, real HW or other options. Some examples are provided here for reference. if you have access to a network device through other mean you can skip this step. Also you can enable multiple networking devices.

=== "containerlab"

    containerlab is a tool that deploys container based meshed lab topologies. Various vendors are supported and extensive documentation is provided here: [containerlab](https://containerlab.srlinux.dev)

    In this example we use SRL for which containerlab sets up gnmi by default
    

=== "real HW"

    Real HW installation should be looked up based on the respective vendor that is used

    E.g. gnmi will be used and should be configured

### Enable connectivity from the cluster to the network device

NDD is using outbound communication from within the kubernetes cluster to the network device. In some environments some special action need to be taken

=== "kind cluster with container lab"

    If you run containerlab on the same linux machine and when both the kind cluster and containerlab uses seperate bridges, you should add the following rules in iptables to allow communication between the kind cluster network and container lab management network.
    The bridges are specific to your environment

    ```markdown
    sudo iptables -I FORWARD 1 -i br-7ee0d52a222c -o br-1f1c85a8b4c5 -j ACCEPT
    sudo iptables -I FORWARD 1 -o br-7ee0d52a222c -i br-1f1c85a8b4c5 -j ACCEPT
    ```


## Install ndd-core

ndd-core provides the core of the ndd project. ndd-core enables:

- the installation, configuration and life cycle management of network device drivers
- the installation, configuration and life cycle management of ndd providers 
- rbac for the ndd providers in the cluster

=== "latest"

    ```
    kubectl apply -f https://raw.githubusercontent.com/yndd/ndd-core/master/config/manifests.yaml
    ```

=== "stable"

    ```
    kubectl apply -f https://raw.githubusercontent.com/yndd/ndd-core/master/config/manifests.yaml
    ```

After this step a ndd-system namespace is created, the ndd-core crds are created and 2 pods are running in ndd-system namespace.

Namespace

```
kubectl get ns
NAME                 STATUS   AGE
default              Active   11m
kube-node-lease      Active   11m
kube-public          Active   11m
kube-system          Active   11m
local-path-storage   Active   11m
ndd-system           Active   2m6s
```

CRDs: 

```
kubectl get crd
NAME                                CREATED AT
controllerconfigs.pkg.ndd.yndd.io   2021-09-05T11:34:20Z
devicedrivers.dvr.ndd.yndd.io       2021-09-05T11:34:20Z
locks.pkg.ndd.yndd.io               2021-09-05T11:34:20Z
networknodes.dvr.ndd.yndd.io        2021-09-05T11:34:20Z
networknodeusages.dvr.ndd.yndd.io   2021-09-05T11:34:20Z
providerrevisions.pkg.ndd.yndd.io   2021-09-05T11:34:20Z
providers.meta.pkg.ndd.yndd.io      2021-09-05T11:34:20Z
providers.pkg.ndd.yndd.io           2021-09-05T11:34:20Z
```

PODs:

```
kubectl get pods -n ndd-system
NAME                        READY   STATUS    RESTARTS   AGE
ndd-core-5dd5b5c679-6hkvw   2/2     Running   0          7m16s
ndd-rbac-6d8d68dbbc-58rnt   2/2     Running   0          7m16s
```

When all of this succeeded we can starrt provisioning network devices through kubernetes