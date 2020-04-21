# Brownfield-Fabric-Automation

## Purpose
Reliably push config to an existing data center comprised of Juniper devices. You can either generate the EVPN/VXLAN configuration and save it in text files (which will require manual push of these config files to your devices. Or you can use these playbooks to dynamically generate and push configs to the devices.

## Requirements
These packages should be installed:
ansible
junos-eznc
Juniper.junos modules from ansible-galaxy

These playbooks were tested on a CentOS-7 device.

## Usage

#### Define the target devices 

###### Define inventory file

All the inventory is defined inside the directory named inventory-dir.
Define lab-inventory in the file named inventory-dir/lab-inventory
Define production-inventory in the file named inventory-dir/production-inventory

This README file assumes that changes are being made to the lab devices (you can execute the same steps and make changes to the production files if you want to execute in a production setting).

Include names of all the leaf-switches under the section-named [lab_leaf], and hostnames of all spine-switches under the section named [lab_spine]

Sample:

```
[lab_spine]
Lab-Spine-1
Lab-Spine-2

[lab_leaf]
Lab-Leaf-1
Lab-Leaf-2
```

###### Define device specific variables

These variables are specific to each device. Example:
-- IP address of each device
-- 4-byte ASNs that may be specific to each Spine
-- Any other variables that you wish to keep specific to each device

*Sample for leaf (file named host_vars/Lab-Leaf-1.yaml):*

```
---
ansible_host: 10.0.0.136
```

*Sample for spine (file named host_vars/Lab-Spine-1.yaml):*

```
---
ansible_host: 10.0.0.135
longASN: 4210011901
```

#### Define parameters needed to generate configuration

This is where you'll define the various VLANs, VNIs, IP addresses etc. These parameters will be used to generate the Junos configuration

*Sample for leaf devices (file named group_vars/lab_leaf.yaml):

```
---
LocalASN: 65001
VLANS:
  - ID: 10
    NAME: VLAN.10
    VLANADDRESS: 10.10.10.100/24
    VGWAdd: 10.10.10.1
    VNI: 10010
  - ID: 20
    NAME: VLAN.20
    VLANADDRESS: 20.20.20.100/24
    VGWAdd: 20.20.20.1
    VNI: 10020
```    
 
 #### Execute the playbook 
 
You can limit the execution based on the role. For example: execute only for Spines, or only for leaf devices.
You can also execute it for an individual switch

*Sample script execution for leaf devices:*
```
ansible-playbook add-evpn-config-leaf.yaml --limit=lab_leaf
```

configuration file with the necessary Junos set commands are placed in the folder called templates/tmp-files/

Output from the above mentioned playbook run:

```
[root@209c9fd40fec Brownfield-Fabric-Automation]# cat templates/tmp-files/Lab-Leaf-1.conf


set protocols igmp-snooping vlan VLAN.10 l2-querier source-address 10.10.10.1
set protocols igmp-snooping vlan VLAN.10 proxy
set vlans VLAN.10  vxlan ingress-node-replication
set interfaces irb unit 10 virtual-gateway-accept-data
set interfaces irb unit 10 description VLAN.10
set interfaces irb unit 10 family inet address 10.10.10.100/24 preferred
set interfaces irb unit 10 family inet address 10.10.10.100/24 virtual-gateway-address 10.10.10.1
set protocols igmp interface irb.10
set protocols evpn vni-options vni 1010 vrf-target target:65001:1010
set protocols evpn extended-vni-list 1010
set policy-options policy-statement OVERLAY_IN term IMPORT_VNI_1010 from community COMM_VNI_1010
set policy-options policy-statement OVERLAY_IN term IMPORT_VNI_1010 then accept
insert policy-options policy-statement OVERLAY_IN term IMPORT_VNI_1010 before term IMPORT_LEAF_ESI
set policy-options community COMM_VNI_1010 members target:65001:1010
set routing-instances OVERLAY_DC interface irb.10
set vlans VLAN.10 vlan-id 10
set vlans VLAN.10 l3-interface irb.10
set vlans VLAN.10 vxlan vni 1010


set protocols igmp-snooping vlan VLAN.20 l2-querier source-address 20.20.20.1
set protocols igmp-snooping vlan VLAN.20 proxy
set vlans VLAN.20  vxlan ingress-node-replication
set interfaces irb unit 20 virtual-gateway-accept-data
set interfaces irb unit 20 description VLAN.20
set interfaces irb unit 20 family inet address 20.20.20.100/24 preferred
set interfaces irb unit 20 family inet address 20.20.20.100/24 virtual-gateway-address 20.20.20.1
set protocols igmp interface irb.20
set protocols evpn vni-options vni 1020 vrf-target target:65001:1020
set protocols evpn extended-vni-list 1020
set policy-options policy-statement OVERLAY_IN term IMPORT_VNI_1020 from community COMM_VNI_1020
set policy-options policy-statement OVERLAY_IN term IMPORT_VNI_1020 then accept
insert policy-options policy-statement OVERLAY_IN term IMPORT_VNI_1020 before term IMPORT_LEAF_ESI
set policy-options community COMM_VNI_1020 members target:65001:1020
set routing-instances OVERLAY_DC interface irb.20
set vlans VLAN.20 vlan-id 20
set vlans VLAN.20 l3-interface irb.20
set vlans VLAN.20 vxlan vni 1020
```
