# OVN-Kubernetes secondary networks for Openshift Virtualization

Read : 

  * https://kubevirt.io/2023/OVN-kubernetes-secondary-networks-policies.html

## Architecture

With OVN-Kubernetes secondary network, you can create additional overlays on top of your cluster nodes.

  * You can specify a private subnet for the new overlay
    
  * You get a DHCP service from OVN-K plugin
    
  * All your pods attached to this overlay can communicate whatever node of the cluster they are hosted on

### OVN-Kubernetes Logical Architecture

[![OVN-Kubernetes Logical Architecture](https://github.com/fdavalo/ocp-virt-ovn-k/blob/main/ovn-k-archi.png?raw=true)](ovn-k-archi.png)

### Secondary network overlays

[![Secondary Network Overlays](https://github.com/fdavalo/ocp-virt-ovn-k/blob/main/secondary-network-overlays-img1.png?raw=true)](secondary-network-overlays-img1.png)

### Workflow Multus <-> OVN-K

[![Workflow Multiple NIC](https://github.com/fdavalo/ocp-virt-ovn-k/blob/main/mult-nic-workflow.png?raw=true)](mult-nic-workflow.png)

## Demo setup

We have 3 Openshift Virtualization VMs,

   * a VM in default namespace
   * two VMs in nico-demo-vm namespace

We declare a NetworkAdditionalDefinition for the OVN-K secondary network : 

           apiVersion: k8s.cni.cncf.io/v1
           kind: NetworkAttachmentDefinition
           metadata:
             name: ovn-k-1
             namespace: default
           spec:
             config: |
               {
                       "cniVersion": "0.3.1", 
                       "name": "ovn-k-1", 
                       "type": "ovn-k8s-cni-overlay", 
                       "topology":"layer2", 
                       "mtu": 1300, 
                       "subnets": "10.100.200.0/24",
                       "netAttachDefName": "default/ovn-k-1" 
               }
    

This network could be declared (with same features) in multiple namespaces) or used from a specific namespace (here the default namespace).

  * Using the same declaration in multiple namespaces, allows management of NetworkAdditionalDefinition only from cluster Admins, and enhance security of those overlays

### See that those NetworkAdditionalDefinition have been added in all 3 VMs : 

[![VM NS default network config](https://github.com/fdavalo/ocp-virt-ovn-k/blob/main/vm-default-config-network.png?raw=true)](vm-default-config-network.png)

[![VM NS nico-demo-vm network config](https://github.com/fdavalo/ocp-virt-ovn-k/blob/main/vm-nico-demo-vm-config-network.png?raw=true)](vm-nico-demo-vm-config-network.png)

### See that those VMs got each an IPs from the overlay subnet : 

[![VM NS default pod ips](https://github.com/fdavalo/ocp-virt-ovn-k/blob/main/vm-default-ips-node.png?raw=true)](vm-default-ips-node.png)

[![VM NS nico-demo-vm pod ips](https://github.com/fdavalo/ocp-virt-ovn-k/blob/main/vm-nico-demo-vm-ips-node.png?raw=true)](vm-nico-demo-vm-ips-node.png)

### See that we can ping/curl from VM in namespace default, the VMs from namespace nico-demo-vm

[![Test Curl between VMs](https://github.com/fdavalo/ocp-virt-ovn-k/blob/main/ping-curl-from-vm.png?raw=true)](ping-curl-from-vm.png)


