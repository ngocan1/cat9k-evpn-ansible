# General description #

In this directory there are playbooks for provisioning DAG (Distributed Anycast Gateway) for Campus EVPN Fabric.

# Playbooks description #

**playbook_underlay.yml:**
- underlay config. Please check the host_vars/xx.yml

**playbook_overlay.yml:** 
- overlay config. Please refer to the overlay configuration parameters

**playbook_output.yml:**  
- executes overlay show commands including the config per node into output/xxx.txt

**playbook_all.yml:**      
- run above three.

**playbook_incremental.yml**
- for partial adding/deleting DAG related config.

# group_vars/all.yml config description #

**OVERLAY CONFIGURATION**

Overlay configuration is mostly the same on all VTEPs. There are some exceptions like unique interfaces/loopbacks or additonal configuration because of device role (Border) or design (for example CGW). For DAG EVPN/NVE related config is the same on all VTEPs, so it is stored in **group_vars/all.yaml** and applicable to all VTEPs.

**global L2VPN parameters**
```
l2vpn_global:
  replication_type: 'static' <<< Configure default replication over multicast for BUM
  router_id: 'Loopback1' <<< Configuring L2VPN router-id
  default_gw: 'yes' <<< Configuring advertising of default gateway
```
**Configuring IP VRF**
```
vrfs:
  green: <<< configuring vrf name
    rd: '1:1' <<< configuring Route-Distinguisher for the respective VRF
    afs:
      ipv4: <<< Configuring IPv4 Address-family
        rt_import: <<< Configuring list of Import Route-Target for IPv4 AF
          - '1:1' 
          - '1:1 stitching'
        rt_export: <<< Configuring list of Export Route-Target for IPv4 AF
          - '1:1'
          - '1:1 stitching'

      ipv6: <<< Configuring IPv6 Address-family
        rt_import: <<< Configuring list of Import Route-Target for IPv6 AF
          - '1:1'
          - '1:1 stitching'
        rt_export: <<< Configuring list of Export Route-Target for IPv6 AF
          - '1:1'
          - '1:1 stitching'
```
**Configuring vlans, evi and vni**
```
vlans:
 101: <<< Configuring VLAN ID
  vlan_type: 'access' <<< Configuring vlan type: "Access" - L2VNI, "Core" - L3VNI, "non_vxlan" - not VXLAN enabled vlan
  description: 'Access_VLAN_101' <<< description of the vlan
  vni: '10101' <<< Configuring VNI to which vlan should be stitched to
  evi: '101'<<< Configuring EVI (Ethernet Virtual Instance) to which vlan should be stichted to

 102:
  vlan_type: 'access'
  description: 'Access_VLAN_102'
  vni: '10102'
  evi: '102'
 
 901:
  vlan_type: 'core'
  description: 'Core_VLAN_VRF_green'
  vni: '50901'
```
**Configuring SVIs**
```
svis:
 101: <<< SVI number
  svi_type: 'access' <<< SVI type: "access"  - for L2VNI, "core" - for L3VNI, "non_vxlan" - for non VXLAN vlans
  vrf: 'green' <<< Configuring VRF
  ipv4: '10.1.101.1 255.255.255.0' <<< Configuring IP Address and Mask

 102:
  svi_type: 'access'
  vrf: 'green'
  ipv4: '10.1.102.1 255.255.255.0'

 901:
  svi_type: 'core' <<< SVI type "core" for L3VNI
  vrf: 'green'
  src_intf: 'Loopback1' <<< Configure "ip unnumbered <interface>" for SVI for L3VNI
  ipv6_enable: 'yes' <<< Enabling IPv6 on SVI
```

**Configuring per EVI parameters**
```
l2vpn_evi:
  101: <<< EVI number
    type: 'vlan-based' <<< type "vlan-based". For now only vlan-based is supprted on cat9k (June 2021)
    encapsulation: 'vxlan' <<< Encapsulation VXLAN. For now only VXLAN encap is supported in cat9k (June 2021)

  102:
    type: 'vlan-based'
    encapsulation: 'vxlan'
```
**Configuring NVI interface** 
```
nve_interfaces:
  1: <<< NVE interface number
    source_interface: 'Loopback1' <<< Configuring source interface for the NVE interface
    vni:
      l2vni: <<< L2VNI configuration part
        10101: <<< L2VNI number
          replication_type: 'static' <<< Configure multicast for the BUM replication
          replication_mcast: '225.0.0.101' <<< Configure multicast group for BUM replication for corresponding VNI
        10102: 
          replication_type: 'ingress-replication' <<< Configure ingress-replication over unicast
      l3vni: <<< L3VNI configuration part
        50901: <<< L3VNI number
          vrf: 'green' <<< Configure the VRF that L3VNI corresponding to
```

**UNDERLAY CONFIGURATION**

Underlay configuration is typically each node specific. The following explains the typical per node (VTEP and SPINE). host_vars/Leaf-xx.yml and host_vars/Spine-xx.yml is where each individual node config is provided. Access interfaces and the vlan's that would be associated with specific access interface is also part of host-vars/Leaf-xx.yml


**Hostname definition**
```
hostname: 'Leaf-01' <<< configuring hostname
```
**Global routing section**
```
routing:
 ipv4_uni: 'yes' <<< enabling ipv4 unicast routing. It must be enabled for basic/advanced routing
 ipv6_uni: 'yes' <<< enabling ipv6 unicast routing. It must be enabled if you have IPv6 uni routing in VRF
 ipv4_multi: 'yes' <<< enabling ipv4 multicast routing. It is needed in case of mcast replication for BUM traffic
```
**Underlay interface configuration**
```
interfaces:

  Loopback0:
    name: 'Routing Loopback' <<< this name will be configured like an description of the interface
    ip_address: '172.16.255.3' <<< configuring ip address 
    subnet_mask: '255.255.255.255' <<< configuring subnet
    loopback: 'yes' <<< is interface a loopback or not
    pim_enable: 'no' <<< enabling|disabling pim on the interface

  Loopback1:
    name: 'NVE Loopback'
    ip_address: '172.16.254.3'
    subnet_mask: '255.255.255.255'
    loopback: 'yes'
    pim_enable: 'yes'

  GigabitEthernet1/0/1:
    name: 'Backbone interface to Spine-01'
    ip_address: '172.16.13.3'
    subnet_mask: '255.255.255.0'
    loopback: 'no'
    pim_enable: 'yes'

  GigabitEthernet1/0/2:
    name: 'Backbone interface to Spine-02'
    ip_address: '172.16.23.3'
    subnet_mask: '255.255.255.0'
    loopback: 'no'
    pim_enable: 'yes' 
```
**Configuring ospf**
```
ospf:
  router_id: '172.16.255.3' <<< configuring router-id for ospf
```
**Configuring PIM RP**
```
pim:
  rp_address: '172.16.255.255' <<< configuring RP address for underlay
```
**Configuring MSDP**
```
msdp:
 '1': <<< just a sequence number of the session
    peer_ip: '172.16.254.2' <<< peering ip address on remote router/switch
    source_interface: 'Loopback1' <<< source interface of the MSDP session
    remote_as: '65001' <<< remote AS number to which 
```
**Configuring BGP**
```
bgp:
  as_number: '65001' <<< Configuring local AS number 
  router_id: 'Loopback0' <<< Configuring BGP router-id

  neighbors:
    '172.16.255.1': <<< Configuring IP address of the neighbor
     peer_as_number: '65001' <<< Configure neighbor AS number
     source_interface: 'Loopback0' <<< Set the source interface for the BGP session for the neighbor

    '172.16.255.2':
     peer_as_number: '65001'
     source_interface: 'Loopback0''
    
    '172.16.255.3':
     peer_as_number: '65001'
     source_interface: 'Loopback0'
     rrc: 'yes' <<< configuring route-reflector client
```
**Overlay Layer3 interfaces**
```
#layer 3 interface
# these layer3 interfaces per vrf, typically useful for dhcp relay source interface
overlay_interfaces:
  Loopback11:
    name: 'Vrf Loopback '
    ip_address: '10.1.11.11'
    subnet_mask: '255.255.255.0'
    loopback: 'yes'
    vrf: 'green'

  Loopback12:
    name: 'Vrf Loopback '
    ip_address: '10.1.12.12'
    subnet_mask: '255.255.255.0'
    loopback: 'yes'
    vrf: 'blue'
```
**Access interfaces**
```
access_interfaces:
  trunks:
    GigabitEthernet1/0/7:
      action: init/add/delete <<< action "init" should be used only during initial interface config. Then add/delete only should be used.
      vlans:
        - 101
        - 102
        - 201
        - 202
```
# Full execution in DAG

It is possible to run all playbooks by using the command (underlay configuration, overlay configuration and show commands):

```
ansible-playbook -i inventory.yaml playbook_all.yml
```

If it is needed to playbook separetly, it is could be run like this:

### Underlay only

```
ansible-playbook -i inventory.yaml playbook_underlay.yml
```

### Overlay only

```
ansible-playbook -i inventory.yaml playbook_overlay.yml
```

### Overlay only

```
ansible-playbook -i inventory.yaml playbook_output.yml
```

# Partial execution in DAG

Full information about the network configuration stored in group_vars/all.yml and host_vars/Leaf-xx.yml(Spine-xx.yml) files.
Those files are "source of truth" for the configuration.
In case of incremental update, new config should be added to the files first.
Then by using one of the below methods only new changes should be provisioned, not full config.

It is alowed to have only part of the configuration to be executed. For example, only new VRF should be added.
Then vrf specific information should be added to **incremental_vars.yml**:

```
 vrf_cli:
    - blue
```
At the same time to the **global_vars/all.yaml** all necessary information should be added:

```
vrfs:
  blue:
    rd: '2:2'
    afs:
      ipv4:
        rt_import: 
          - '2:2'
          - '2:2 stitching'
        rt_export: 
          - '2:2'
          - '2:2 stitching'
      ipv6:
        rt_import: 
          - '2:2'
          - '2:2 stitching'
        rt_export: 
          - '2:2'
          - '2:2 stitching'
```

There are 3 option to trigger partial execution of the configuration:
- adding external variables in the command line 
- defining variables in partial_execution.yaml file
- defining variables in playbook directly

## Variable options

- **vrf_cli**           define the vrfs
- **vlan_cli**          define the vlan list
- **svi_cli**           define the svi list
- **l2vpn_evi_cli**     define the EVI list
- **ovrl_intf_cli**     define the Overlay interfaces list

## External variables in the command line

Example

```
ansible-playbook -i inventory.yaml playbook_overlay.yml -v -e "vrf_cli=['green']"
```

## Variables in partial_execution.yaml file

```
    vrf_cli:
      - green
      
    vlan_cli:
      - 201
      - 202
      - 902

    svi_cli:
      - 201
      - 202
      - 902

    l2vpn_evi_cli:
      - 201
      - 202

    ovrl_intf_cli:
      - Loopback11
```

## Variables in playbook directly

```
---

- name: Automated VXLAN deployment with BGP EVPN L2/L3 underlay w/ Spine
  hosts: all
  gather_facts: no
  vars:
    partial_run: false

    vrf_cli: 
      - green
```
