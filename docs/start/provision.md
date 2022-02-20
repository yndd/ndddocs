# Provision

NDD allow to provision a network device through kubernetes using a provider. 

The provider interacts with the device through GNMI and implements a caching layer for robust operations between kubernetes and the network device. 

The provider installs the configuration parameters in a declaritive way to the network device and ensure the k8s crds are synchronized with the device.

## Install the Provider

Install a provider which exposes the network device configuration through the kubernetes API.

Create a  nddp-package-srl.yaml file, which provides:
- name of the provider: nddp-srl
- package name: yndd/nddp-srl:latest

file nddp-package-srl.yaml

```
apiVersion: pkg.ndd.yndd.io/v1
kind: Provider
metadata:
  name: nddp-srl
  namespace: ndd-system
spec:
  package: yndd/nddp-srl:latest
  packagePullPolicy: Always
```

apply the provider to the cluster

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
nddp-srl-51ef00fd88c2-98fb57465-lm5td            2/2     Running   0          4m3s
ndd-rbac-6d8d68dbbc-58rnt                        2/2     Running   0          51m
```

Also you should see a number of CRDs being installed in the cluster which are owned by the provider you installed.

```
kubectl get crd | grep srl
```

```
srlbfds.srl.nddp.yndd.io                                                    2022-02-09T14:48:28Z
srlinterfaces.srl.nddp.yndd.io                                              2022-02-09T14:48:28Z
srlinterfacesubinterfaces.srl.nddp.yndd.io                                  2022-02-09T14:48:28Z
srlnetworkinstanceaggregateroutes.srl.nddp.yndd.io                          2022-02-09T14:48:28Z
srlnetworkinstancenexthopgroups.srl.nddp.yndd.io                            2022-02-09T14:48:28Z
srlnetworkinstanceprotocolsbgpevpns.srl.nddp.yndd.io                        2022-02-09T14:48:28Z
srlnetworkinstanceprotocolsbgps.srl.nddp.yndd.io                            2022-02-09T14:48:28Z
srlnetworkinstanceprotocolsbgpvpns.srl.nddp.yndd.io                         2022-02-09T14:48:28Z
srlnetworkinstanceprotocolsises.srl.nddp.yndd.io                            2022-02-09T14:48:28Z
srlnetworkinstanceprotocolslinuxes.srl.nddp.yndd.io                         2022-02-09T14:48:28Z
srlnetworkinstanceprotocolsospfs.srl.nddp.yndd.io                           2022-02-09T14:48:28Z
srlnetworkinstances.srl.nddp.yndd.io                                        2022-02-09T14:48:28Z
srlnetworkinstancestaticroutes.srl.nddp.yndd.io                             2022-02-09T14:48:28Z
srlroutingpolicyaspathsets.srl.nddp.yndd.io                                 2022-02-09T14:48:28Z
srlroutingpolicycommunitysets.srl.nddp.yndd.io                              2022-02-09T14:48:28Z
srlroutingpolicypolicies.srl.nddp.yndd.io                                   2022-02-09T14:48:29Z
srlroutingpolicyprefixsets.srl.nddp.yndd.io                                 2022-02-09T14:48:29Z
srlsystemnames.srl.nddp.yndd.io                                             2022-02-09T14:48:29Z
srlsystemnetworkinstanceprotocolsbgpvpns.srl.nddp.yndd.io                   2022-02-09T14:48:29Z
srlsystemnetworkinstanceprotocolsevpnesisbgpinstanceesis.srl.nddp.yndd.io   2022-02-09T14:48:29Z
srlsystemnetworkinstanceprotocolsevpnesisbgpinstances.srl.nddp.yndd.io      2022-02-09T14:48:29Z
srlsystemnetworkinstanceprotocolsevpns.srl.nddp.yndd.io                     2022-02-09T14:48:29Z
srlsystemntps.srl.nddp.yndd.io                                              2022-02-09T14:48:29Z
srltransactions.srl.nddp.yndd.io                                            2022-02-13T20:38:49Z
srltunnelinterfaces.srl.nddp.yndd.io                                        2022-02-09T14:48:29Z
srltunnelinterfacevxlaninterfaces.srl.nddp.yndd.io                          2022-02-09T14:48:29Z
```
## Setup a network node device driver

To setup the network node device we first need to configure a network node.

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

When the provider and the network device type matches you should see the network node becoming in ready status and you should see that the network node got discovered with additional parameters of the network device, such as sw version ,type, etc

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
<<<<<<< HEAD
apiVersion: srl.ndd.yndd.io/v1alpha1
=======
apiVersion: srl.nddp.yndd.io/v1alpha1
>>>>>>> d896e49 (added some new docs)
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
