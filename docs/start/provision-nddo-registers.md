# Provision

First we need to configure the registries

## Configure the organization registry

### organization

we create an organization 

```
apiVersion: org.nddr.yndd.io/v1alpha1
kind: Organization
metadata:
  name: nokia
  namespace: default
spec:
  organization:
    description: default organization for Nokia
    register:
    - {kind: ipam, name: nokia-default}
    - {kind: ni, name: nokia-default}
    - {kind: as, name: nokia-default}
    - {kind: vlan, name: nokia-default}
```

### deployment

create a deployment

```
apiVersion: org.nddr.yndd.io/v1alpha1
kind: Deployment
metadata:
  name: nokia.region1
  namespace: default
spec:
  deployment:
    region: an
```

## Configure a topology in a deployment

### fabric

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: Topology
metadata:
  name: nokia.region1.fabric1
  namespace: default
spec:
  topology:
    kind:
    - name: srl
      tag:
      - key: platform
        value: ixrd2
    - name: sros
      tag:
      - key: platform
        value: sr1
    - name: linux
```

### nodes

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyNode
metadata:
  name: nokia.region1.fabric1.leaf1
  namespace: default
spec:
  node:
    kind-name: srl
    tag:
    - {key: index, value: "0"}
    - {key: position, value: leaf}
```

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyNode
metadata:
  name: nokia.region1.fabric1.leaf2
  namespace: default
spec:
  node:
    kind-name: srl
    tag:
    - {key: index, value: "1"}
    - {key: position, value: leaf}
```

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyNode
metadata:
  name: nokia.region1.fabric1.dcgw1
  namespace: default
spec:
  node:
    kind-name: sros
    tag:
    - {key: position, value: dcgw}
```

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyNode
metadata:
  name: nokia.region1.fabric1.dcgw2
  namespace: default
spec:
  node:
    kind-name: sros
    tag:
    - {key: position, value: dcgw}
```

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyNode
metadata:
  name: nokia.region1.fabric1.master0
  namespace: default
spec:
  node:
    kind-name: linux
    tag:
    - {key: position, value: server}
```

### links

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyLink
metadata:
  name: nokia.region1.fabric1.link50-leaf1-leaf2
  namespace: default
spec:
  link:
    tag:
    - {key: lag-member, value: "true"}
    endpoints:
    - node-name: leaf1
      interface-name: int-1/1/50
      tag:
      - {key: kind, value: infra}
      - {key: lag-name, value: lag-50}
    - node-name: leaf2
      interface-name: int-1/1/50
      tag:
      - {key: kind, value: infra}
      - {key: lag-name, value: lag-50}
```

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyLink
metadata:
  name: nokia.region1.fabric1.link49-leaf1-leaf2
  namespace: default
spec:
  link:
    tag:
    - {key: lag-member, value: "true"}
    endpoints:
    - node-name: leaf1
      interface-name: int-1/1/49
      tag:
      - {key: kind, value: infra}
      - {key: lag-name, value: lag-50}
    - node-name: leaf2
      interface-name: int-1/1/49
      tag:
      - {key: kind, value: infra}
      - {key: lag-name, value: lag-50}
```

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyLink
metadata:
  name: nokia.region1.fabric1.link48-leaf1-dcgw1
  namespace: default
spec:
  link:
    tag:
    endpoints:
    - node-name: leaf1
      interface-name: int-1/1/48
      tag:
      - {key: kind, value: access}
      - {key: endpoint-group, value: dcgw}
    - node-name: dcgw1
      interface-name: int-1/1/1
      tag:
      - {key: kind, value: access}
```

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyLink
metadata:
  name: nokia.region1.fabric1.link48-leaf2-dcgw2
  namespace: default
spec:
  link:
    tag:
    endpoints:
    - node-name: leaf2
      interface-name: int-1/1/48
      tag:
      - {key: kind, value: access}
      - {key: endpoint-group, value: dcgw}
    - node-name: dcgw2
      interface-name: int-1/1/1
      tag:
      - {key: kind, value: access}
```

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyLink
metadata:
  name: nokia.region1.fabric1.link1-leaf2-master0
  namespace: default
spec:
  link:
    tag:
    - {key: lag-member, value: "true"}
    endpoints:
    - node-name: leaf2
      interface-name: int-1/1/1
      tag:
      - {key: kind, value: access}
      - {key: lag-name, value: lag-1}
      - {key: lacp-fallback, value: "true"}
      - {key: endpoint-group, value: server-pod1}
      - {key: multihoming, value: "true"}
      - {key: multihoming-name, value: esi1}
    - node-name: master0
      interface-name: ens0
      tag:
      - {key: kind, value: access}
      - {key: lag-name, value: bond0}
```

```
apiVersion: topo.nddr.yndd.io/v1alpha1
kind: TopologyLink
metadata:
  name: nokia.region1.fabric1.link1-leaf1-master0
  namespace: default
spec:
  link:
    tag:
    - {key: lag-member, value: "true"}
    endpoints:
    - node-name: leaf1
      interface-name: int-1/1/1
      tag:
      - {key: kind, value: access}
      - {key: lag-name, value: lag-1}
      - {key: lacp-fallback, value: "true"}
      - {key: endpoint-group, value: server-pod1}
      - {key: multihoming, value: "true"}
      - {key: multihoming-name, value: esi1}
    - node-name: master0
      interface-name: ens0
      tag:
      - {key: kind, value: access}
      - {key: lag-name, value: bond0}

```

The status of the links can be shown like this

```
k get topologylinks.topo.nddr.yndd.io
```

```
NAME                                                              SYNC   STATUS   ORG     DEP       AZ    TOPO      LAG    MEMBER   NODE-EPA   ITFCE-EPA    MH-EPA   NODE-EPB   ITFCE-EPB    MH-EPB   AGE
nokia.region1.fabric1.link1-leaf1-master0                         True   True     nokia   region1         fabric1          true     leaf1      int-1/1/1    true     master0    ens0                  11m
nokia.region1.fabric1.link1-leaf2-master0                         True   True     nokia   region1         fabric1          true     leaf2      int-1/1/1    true     master0    ens0                  11m
nokia.region1.fabric1.link48-leaf1-dcgw1                          True   True     nokia   region1         fabric1                   leaf1      int-1/1/48            dcgw1      int-1/1/1             11m
nokia.region1.fabric1.link48-leaf2-dcgw2                          True   True     nokia   region1         fabric1                   leaf2      int-1/1/48            dcgw2      int-1/1/1             11m
nokia.region1.fabric1.link49-leaf1-leaf2                          True   True     nokia   region1         fabric1          true     leaf1      int-1/1/49            leaf2      int-1/1/49            11m
nokia.region1.fabric1.link50-leaf1-leaf2                          True   True     nokia   region1         fabric1          true     leaf1      int-1/1/50            leaf2      int-1/1/50            11m
nokia.region1.fabric1.logical-mh-link-esi1-master0                True   True     nokia   region1         fabric1   true                       esi1         true     master0    bond0                 11m
nokia.region1.fabric1.logical-sh-link-leaf1-lag-50-leaf2-lag-50   True   True     nokia   region1         fabric1   true            leaf1      lag-50                leaf2      lag-50                11m

```

## Configure the ni registry

The ni registry helps to allocate unique identifiers for the vpc when using irb constructs and are used for vxlan and irb identifiers

```
apiVersion: ni.nddr.yndd.io/v1alpha1
kind: Registry
metadata:
  name: nokia-default
  namespace: default
spec:
  oda:
  - key: organization
    value: nokia
  registry:
    description: default Network Instance pool
    size: 10000
```

show the registry status

```
k get registries.ni.nddr.yndd.io
```

```
NAME            SYNC   STATUS   ORG     DEP       AZ        REGISTRY        ALLOCATED   AVAILABLE   TOTAL   AGE
nokia-default   True   True     nokia   unknown   unknown   nokia-default   0           10000       10000   23s
```