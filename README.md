# Simple ovn-kubernetes Chart

This repository only includes a basic Helm Chart for [ovn-kubernetes](https://ovn-kubernetes.io/).

The official Helm Charts can be found in the official repository: 
<https://github.com/ovn-kubernetes/ovn-kubernetes/tree/master/helm/ovn-kubernetes>. 
This repository focuses instead on providing a simpler chart, with instructions that can
simplify the deployment of the ovn-kubernetes CNI on [Canonical k8s](https://github.com/canonical/k8s-snap)

## Node configuration

Some configurations are required on the Kubernetes nodes before the ovn-kubernetes CNI
can be installed.

1. An OVS bridge must be defined for the interface used for the cluster, for example using `netplan`:

```yaml
network:
    ethernets:
        eth0:
            dhcp4: true
        eth1:
            dhcp4: false
    bridges:
        br0:
            interfaces: [eth1]
            addresses:
                - "<node-ip>/24"
            routes:
            - to: "0.0.0.0/0"
              via: "<gateway-ip>"
            nameservers:
                addresses:
                - "<dns-ip>"
            parameters:
                forward-delay: "15"
                stp: false
            openvswitch: {}
    version: 2
```

2. Configure OVS with the informatio about this bridge and the NIC used as uplink

```bash
$ sudo ovs-vsctl br-set-external-id br0 bridge-id br0 -- br-set-external-id br0 bridge-uplink enp8s0
```

3. Label the node to use its hostname as `zone-name`

```bash
$ nodename=$(kubectl get nodes -o json | jq -r .items[0].metadata.name)
$ kubectl label node --overwrite $nodename k8s.ovn.org/zone-name=$nodename
```

4. Create the `ovn-kubernetes` namespace

```bash
$ kubectl create ns ovn-kubernetes
```

## Install ovn-kubernetes CNI

While the `ovn-kubernetes` chart provides numerous configurations, this chart can be
deployed providing simply the API server URL.

```bash
$ helm install  -n ovn-kubernetes ovn-kubernetes . --set k8sAPIServer=https://<API_SERVER_IP>:6443
```
