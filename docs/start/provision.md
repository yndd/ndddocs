# Provision

NDD allow to provision a network device through kubernetes using a network node device (nn) driver and a ndd-provider. 

The network node device driver interacts with the device through GNMI and provides a caching layer between the network device and the provider.

The ndd-provider installs the configuration parameters in a declaritive way to the network device, through the network node device driver.

## Setup a network node device driver

To setup the network node device driver we first need to configure a network node and optionally device driver parameters.

### Setup a secret

A secret is used to connect to the network device through gnmi.

Create a nddsrlsecret.yaml file with the base64 encoded username and password

file: nddsrlsecret.yaml

```
---
apiVersion: v1
kind: Secret
metadata:
  name: srl-secrets
type: Opaque
data:
  username: YWRtaW4K
  password: YWRtaW4K
```

apply the secret to the cluster

```
kubectl apply -f nddsrlsecret.yaml
secret/srl-secrets created
```

Create a network node file leaf1.yaml:

- in the credentialsName: refer to the name of the secret created in the previous step
- in the address: supply the IP address of the network node that was setup in the network node installation step. In this Example we use gnmi between the network device driver and the network device, so also the gnmi port is relevant in the address section.

file leaf1.yaml

```
---
apiVersion: dvr.ndd.yndd.io/v1
kind: NetworkNode
metadata:
  name: leaf1
  labels:
    target-group: leaf-grp1
    target: leaf1
spec:
#  deviceDriverKind: gnmi
  target:
    address: 172.20.20.5:57400
    credentialsName: srl-secrets
    encoding: JSON_IETF
    skpVerify: true
```

apply the network node configuration to the cluster

```
 kubectl apply -f leaf1.yaml
networknode.dvr.ndd.yndd.io/leaf1 unchanged
```

After the creation of the network node we should see:

- a new pod appear in the cluster which is providing the caching function between the network device and the ndd-provider
- the network node state should be HEALTHY: True, CONFIGURED: True, READY: False. READY state is false because the provider is not yet installed and registered to the network node device driver.

Pods in ndd-system namespace

```
kubectl  get pods -n ndd-system
```

```
NAME                             READY   STATUS    RESTARTS   AGE
ndd-core-5dd5b5c679-6hkvw        2/2     Running   0          38m
ndd-dep-leaf1-5bf7b6b89f-ncd8x   1/1     Running   0          17m
ndd-rbac-6d8d68dbbc-58rnt        2/2     Running   0          38m
```

Network node status:

```
kubectl get nn
```

```
NAME    HEALTHY   CONFIGURED   READY   ADDRESS             CONN-KIND   TYPE   KIND   SWVERSION   MACADDRESS   SERIALNBR   GRPCSERVERPORT   AGE
leaf1   True      True         False   172.20.20.5:57400   gnmi                                                           9999             19m
```

## Install the Provider

Install a provider which exposes the network device configuration through the kubernetes API.

Create a nddsrlpackage.yaml file, which provides:
- name of the provider: ndd-provider-srl
- package name: yndd/ndd-provider-srl:latest

file nddsrlpackage.yaml

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Provider
metadata:
  name: ndd-provider-srl
  namespace: ndd-system
spec:
  package: yndd/ndd-provider-srl:latest
  packagePullPolicy: Always
```

apply the provider to the cluster

```
kubectl apply -f nddsrlsecret.yaml
secret/srl-secrets created
```

to see the installation of the provider you can use:

```
watch kubectl get pkg
```

After this step is successfull you should see the provider pod running in the ndd-system namespace

```
kubectl get pods -n ndd-system
```

```
NAME                                             READY   STATUS    RESTARTS   AGE
ndd-core-5dd5b5c679-6hkvw                        2/2     Running   0          51m
ndd-dep-leaf1-5bf7b6b89f-ncd8x                   1/1     Running   0          30m
ndd-provider-srl-67b9e61445f6-846cb5c64c-t2w9h   1/1     Running   0          2m35s
ndd-rbac-6d8d68dbbc-58rnt                        2/2     Running   0          51m
```

Also you should see a number of CRDs being installed in the cluster which are owned by the provider you installed.

```
kubectl get crd | grep srl
```

```
registrations.srl.ndd.yndd.io                                              2021-09-05T12:23:01Z
srlbfds.srl.ndd.yndd.io                                                    2021-09-05T12:23:01Z
srlinterfaces.srl.ndd.yndd.io                                              2021-09-05T12:23:01Z
srlinterfacesubinterfaces.srl.ndd.yndd.io                                  2021-09-05T12:23:01Z
srlnetworkinstanceaggregateroutes.srl.ndd.yndd.io                          2021-09-05T12:23:01Z
srlnetworkinstancenexthopgroups.srl.ndd.yndd.io                            2021-09-05T12:23:01Z
srlnetworkinstanceprotocolsbgpevpns.srl.ndd.yndd.io                        2021-09-05T12:23:01Z
srlnetworkinstanceprotocolsbgps.srl.ndd.yndd.io                            2021-09-05T12:23:01Z
srlnetworkinstanceprotocolsbgpvpns.srl.ndd.yndd.io                         2021-09-05T12:23:01Z
srlnetworkinstanceprotocolsises.srl.ndd.yndd.io                            2021-09-05T12:23:01Z
srlnetworkinstanceprotocolslinuxes.srl.ndd.yndd.io                         2021-09-05T12:23:01Z
srlnetworkinstanceprotocolsospfs.srl.ndd.yndd.io                           2021-09-05T12:23:01Z
srlnetworkinstances.srl.ndd.yndd.io                                        2021-09-05T12:23:01Z
srlnetworkinstancestaticroutes.srl.ndd.yndd.io                             2021-09-05T12:23:01Z
srlroutingpolicyaspathsets.srl.ndd.yndd.io                                 2021-09-05T12:23:01Z
srlroutingpolicycommunitysets.srl.ndd.yndd.io                              2021-09-05T12:23:01Z
srlroutingpolicypolicies.srl.ndd.yndd.io                                   2021-09-05T12:23:02Z
srlroutingpolicyprefixsets.srl.ndd.yndd.io                                 2021-09-05T12:23:02Z
srlsystemmtus.srl.ndd.yndd.io                                              2021-09-05T12:23:02Z
srlsystemnames.srl.ndd.yndd.io                                             2021-09-05T12:23:02Z
srlsystemnetworkinstanceprotocolsbgpvpns.srl.ndd.yndd.io                   2021-09-05T12:23:02Z
srlsystemnetworkinstanceprotocolsevpnesisbgpinstanceesis.srl.ndd.yndd.io   2021-09-05T12:23:02Z
srlsystemnetworkinstanceprotocolsevpnesisbgpinstances.srl.ndd.yndd.io      2021-09-05T12:23:02Z
srlsystemnetworkinstanceprotocolsevpns.srl.ndd.yndd.io                     2021-09-05T12:23:02Z
srlsystemntps.srl.ndd.yndd.io                                              2021-09-05T12:23:02Z
srltunnelinterfaces.srl.ndd.yndd.io                                        2021-09-05T12:23:02Z
srltunnelinterfacevxlaninterfaces.srl.ndd.yndd.io                          2021-09-05T12:23:02Z
```

When the provider and the network device type matches you should see the network node becoming in ready status and you should see that the network node discovered additional parameters of the network device, such as sw version ,type, etc

```
kubectl get nn
```

```
NAME    HEALTHY   CONFIGURED   READY   ADDRESS             CONN-KIND   TYPE        KIND          SWVERSION   MACADDRESS          SERIALNBR        GRPCSERVERPORT   AGE
leaf1   True      True         True    172.20.20.5:57400   gnmi        nokia-srl   7220 IXR-D2   v21.3.1     02:BF:A9:FF:00:00   Sim Serial No.   9999             47m
```

## Provision a NDD managed resource

Create a managed resource for the provider installed. E.g. an interface and subinterface.
These resources are specific for the ndd provider you use.

A ndd managed resource has the following attributes:
- Kubernetes specific parameters:
    - A apiversion: enabled by the provider
    - A Kind: enabled by the provider
    - Metadata: name, namespace, label and annotations
- Spec specifics for ndd:
    - networkNodeRef: refers to the network node name that was created in the previous step
    - forNetworkNode: refers to the specifics of the managed resource CRD enabled by the provider: see provider API documentation for more details.

file int-e1-49.yaml

```
apiVersion: srl.ndd.yndd.io/v1alpha1
kind: SrlInterface
metadata:
  name: int-e1-49
  namespace: default
spec:
  active: true
  networkNodeRef: 
    name: leaf1
  interface:
    name: "ethernet-1/49"
    admin-state: "enable"
    description: "ndd-ethernet-1/49"
    vlan-tagging: true
```

```
kubectl apply -f int-e1-49.yaml
```

file subint-e1-49-0.yaml 

```
apiVersion: srl.nddp.yndd.io/v1alpha1
kind: SrlInterfaceSubinterface
metadata:
  name: subint-e1-49-0
  namespace: default
spec:
  active: true
  networkNodeRef:
    name: leaf1
  interface-name: ethernet-1/49
  subinterface:
      index: 1
      type: routed
      admin-state: enable
      description: "ndd-e1-49-0-leaf1"
      ipv4:
        address:
        - ip-prefix: 100.64.0.0/31
      ipv6:
        address:
        - ip-prefix: 3100:64::/127
      vlan:
        encap:
          single-tagged:
            vlan-id: "1"

```

```
kubectl apply -f subint-e1-49-0.yaml 
```

When loggind in to the node you should see the configuration that was supplied via the ndd-provider in the network device

```
info from running / interface ethernet-1/49
```

```
    interface ethernet-1/49 {
        description ndd-ethernet-1/49
        admin-state enable
        vlan-tagging true
        subinterface 1 {
            type routed
            description ndd-e1-49-0-leaf1
            admin-state enable
            ipv4 {
                allow-directed-broadcast false
                address 100.64.0.0/31 {
                }
            }
            ipv6 {
                address 3100:64::/127 {
                }
            }
            vlan {
                encap {
                    single-tagged {
                        vlan-id 1
                    }
                }
            }
        }
    }
```
