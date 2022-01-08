# Provision

After the registers are setup we can configure the intents

## infrastructure

```
apiVersion: infra.nddo.yndd.io/v1alpha1
kind: Infrastructure
metadata:
  name: nokia.region1.infra
  namespace: default
spec:
  infrastructure:
    network-instance-name: default-routed
    admin-state: enable
    addressing-scheme: dual-stack
    underlay-protocol:
    - 'ebgp'
    overlay-protocol:
    - 'evpn'

```

show status

```
k get infrastructures.infra.nddo.yndd.io
NAME                  SYNC   STATUS   ORG     DEP       AZ    NI               ADDR         UNDERLAY   OVERLAY   AGE
nokia.region1.infra   True   True     nokia   region1         default-routed   dual-stack   ebgp       evpn      23s
```

this should result in the network instances and interfaces created in the adaptor layer

```
k get interfaces.network.ndda.yndd.io
NAME                                                            AGE
nokia.region1.infrastructure.infra.leaf1-int-1-1-49-interface   94s
nokia.region1.infrastructure.infra.leaf1-int-1-1-50-interface   94s
nokia.region1.infrastructure.infra.leaf1-irb-irb                94s
nokia.region1.infrastructure.infra.leaf1-lag-50-interface       94s
nokia.region1.infrastructure.infra.leaf1-system-system          94s
nokia.region1.infrastructure.infra.leaf1-vxlan-vxlan            94s
nokia.region1.infrastructure.infra.leaf2-int-1-1-49-interface   94s
nokia.region1.infrastructure.infra.leaf2-int-1-1-50-interface   94s
nokia.region1.infrastructure.infra.leaf2-irb-irb                94s
nokia.region1.infrastructure.infra.leaf2-lag-50-interface       94s
nokia.region1.infrastructure.infra.leaf2-system-system          94s
nokia.region1.infrastructure.infra.leaf2-vxlan-vxlan            94s
```

```
k get networkinstances.network.ndda.yndd.io
NAME                                       AGE
nokia.region1.infrastructure.infra.leaf1   108s
nokia.region1.infrastructure.infra.leaf2   108s
```

## endpoint groups

server pods

```
apiVersion: epg.nddo.yndd.io/v1alpha1
kind: EpGroup
metadata:
  name: nokia.region1.server-pod1
  namespace: default
spec:
  ep-group:
    admin-state: enable
```

dcgw

```
apiVersion: epg.nddo.yndd.io/v1alpha1
kind: EpGroup
metadata:
  name: nokia.region1.dcgw
  namespace: default
spec:
  ep-group:
    admin-state: enable
```

show status

```
k get epgroups.epg.nddo.yndd.io
NAME                        SYNC   STATUS   ORG     DEP       AZ    EPGNAME       DEPL   AGE
nokia.region1.dcgw          True   True     nokia   region1         dcgw                 72s
nokia.region1.server-pod1   True   True     nokia   region1         server-pod1          72s
```

This shoudl result in additional interfaces created in the adaptor layer

```
 k get interfaces.network.ndda.yndd.io
NAME                                                            AGE
nokia.region1.epgroup.dcgw.leaf1-int-1-1-48-interface           118s
nokia.region1.epgroup.dcgw.leaf2-int-1-1-48-interface           118s
nokia.region1.epgroup.server-pod1.leaf1-int-1-1-1-interface     118s
nokia.region1.epgroup.server-pod1.leaf1-lag-1-interface         118s
nokia.region1.epgroup.server-pod1.leaf2-int-1-1-1-interface     118s
nokia.region1.epgroup.server-pod1.leaf2-lag-1-interface         118s
```

## vpc

```
apiVersion: vpc.nddo.yndd.io/v1alpha1
kind: Vpc
metadata:
  name: nokia.region1.prov
  namespace: default
spec:
  vpc:
    admin-state: enable
    description: vpc for server provisioning
    bridge-domains:
    - name: provisioning
      interface-selector:
      - tag:
          - {key: kind, value: epg}
          - {key: endpoint-group, value: server-pod1}
        vlan: 10
```

```
apiVersion: vpc.nddo.yndd.io/v1alpha1
kind: Vpc
metadata:
  name: nokia.region1.infra
  namespace: default
spec:
  vpc:
    defaults:
      tunnel: vxlan
      protocol: evpn
    admin-state: enable
    description: vpc for server infrastructure
    bridge-domains:
    - name: infrastructure
      tunnel: vxlan
      protocol: evpn
      interface-selector:
      - tag:
        - {key: kind, value: epg}
        - {key: endpoint-group, value: server-pod1}
        vlan: 40
    routing-tables:
    - name: infrastructure
      tunnel: vxlan
      protocol: evpn
      bridge-domains:
      - name: infrastructure
        ipv4-prefixes: [100.112.3.0/24]
        ipv6-prefixes: [2a02:1800:80:7000::/64]
      interface-selector:
      - tag:
        - {key: kind, value: node-itfce}
        - {key: leaf1, value: int-1/1/48}
        ipv4-prefixes: [100.112.10.1/31]
        ipv6-prefixes: [2a02:1800:80:7050::1/64]
        vlan: 10
      - tag:
        - {key: kind, value: node-itfce}
        - {key: leaf2, value: int-1/1/48}
        ipv4-prefixes: [100.112.10.3/31]
        ipv6-prefixes: [2a02:1800:80:7051::1/64]
        vlan: 11
```

show status

```
k get vpcs.vpc.nddo.yndd.io
NAME                  SYNC   STATUS   ORG     DEP       AZ    VPC             AGE
nokia.region1.infra   True   True     nokia   region1         infra           8s
nokia.region1.prov    True   True     nokia   region1         prov            13s
```

this shoudl result in additional network instances and subinterface creation in the adaptor layer

```
k get subinterfaces.network.ndda.yndd.io
NAME                                                       AGE
nokia.region1.infrastructure.infra.leaf1-lag-50-0-routed   6m57s
nokia.region1.infrastructure.infra.leaf1-system-0-routed   6m57s
nokia.region1.infrastructure.infra.leaf2-lag-50-0-routed   6m57s
nokia.region1.infrastructure.infra.leaf2-system-0-routed   6m57s
nokia.region1.vpc.infra.leaf1-int-1-1-48-10-routed         98s
nokia.region1.vpc.infra.leaf1-irb-12303-routed             98s
nokia.region1.vpc.infra.leaf1-irb-2303-bridged             98s
nokia.region1.vpc.infra.leaf1-lag-1-40-bridged             98s
nokia.region1.vpc.infra.leaf1-vxlan-2241-routed            98s
nokia.region1.vpc.infra.leaf1-vxlan-2303-bridged           98s
nokia.region1.vpc.infra.leaf2-int-1-1-48-11-routed         98s
nokia.region1.vpc.infra.leaf2-irb-12303-routed             98s
nokia.region1.vpc.infra.leaf2-irb-2303-bridged             98s
nokia.region1.vpc.infra.leaf2-lag-1-40-bridged             98s
nokia.region1.vpc.infra.leaf2-vxlan-2241-routed            98s
nokia.region1.vpc.infra.leaf2-vxlan-2303-bridged           98s
nokia.region1.vpc.prov.leaf1-lag-1-10-bridged              103s
nokia.region1.vpc.prov.leaf1-vxlan-2085-bridged            103s
nokia.region1.vpc.prov.leaf2-lag-1-10-bridged              103s
nokia.region1.vpc.prov.leaf2-vxlan-2085-bridged            103s
```

```
k get networkinstances.network.ndda.yndd.io
NAME                                       AGE
nokia.region1.infrastructure.infra.leaf1   7m18s
nokia.region1.infrastructure.infra.leaf2   7m18s
nokia.region1.vpc.infra.leaf1              119s
nokia.region1.vpc.infra.leaf2              119s
nokia.region1.vpc.prov.leaf1               2m4s
nokia.region1.vpc.prov.leaf2               2m4s
```