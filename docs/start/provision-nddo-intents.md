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
    network-instance-name: default
    as: 65555
    as-pool:
      start: 65000
      end: 65400
    cidr:
      loopback-cidr-ipv4: "100.64.0.0/24"
      loopback-cidr-ipv6: "1000:64::/64"
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

this should result in the network instances and interfaces created in the nddp-layer

```
k get srl2

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
 k get srl2
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
        outer-vlan-id: 10
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
        outer-vlan-id: 40
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
        outer-vlan-id: 10
      - tag:
        - {key: kind, value: node-itfce}
        - {key: leaf2, value: int-1/1/48}
        ipv4-prefixes: [100.112.10.3/31]
        ipv6-prefixes: [2a02:1800:80:7051::1/64]
        outer-vlan-id: 11
```

```
apiVersion: vpc.nddo.yndd.io/v1alpha1
kind: Vpc
metadata:
  name: nokia.region1.multus
  namespace: default
spec:
  vpc:
    defaults:
      tunnel: vxlan
      protocol: evpn
    admin-state: enable
    description: vpc for multus
    bridge-domains:
    - name: multus-ipvlan
      interface-selector:
      - tag:
        - {key: kind, value: epg}
        - {key: endpoint-group, value: server-pod1}
        - {key: itfce-kind, value: ipvlan}
        outer-vlan-id: 100
    - name: multus-sriov4
      interface-selector:
      - tag:
        - {key: kind, value: epg}
        - {key: endpoint-group, value: server-pod1}
        - {key: itfce-kind, value: sriov1}
        outer-vlan-id: 200
    - name: multus-sriov55
      interface-selector:
      - tag:
        - {key: kind, value: epg}
        - {key: endpoint-group, value: server-pod1}
        - {key: itfce-kind, value: sriov2}
        outer-vlan-id: 102
    routing-tables:
    - name: multus
      bridge-domains:
      - name: multus-ipvlan2
        ipv4-prefixes: [100.112.1.3/24]
        ipv6-prefixes: [2a02:1800:81:8000::/64]
      - name: multus-sriov12
        ipv4-prefixes: [100.112.2.0/24]
        ipv6-prefixes: [2a02:1800:82:7000::/64]
      - name: multus-sriov2
        ipv4-prefixes: [100.112.3.0/24]
        ipv6-prefixes: [2a02:1800:83:7000::/64]
      interface-selector:
      - tag:
        - {key: kind, value: node-itfce}
        - {key: leaf1, value: int-1/1/48}
        ipv4-prefixes: [100.112.10.1/31]
        ipv6-prefixes: [2a02:1800:80:7050::1/64]
        outer-vlan-id: 100
      - tag:
        - {key: kind, value: node-itfce}
        - {key: leaf2, value: int-1/1/48}
        ipv4-prefixes: [100.112.10.3/31]
        ipv6-prefixes: [2a02:1800:80:7050::3/64]
        outer-vlan-id: 110

```