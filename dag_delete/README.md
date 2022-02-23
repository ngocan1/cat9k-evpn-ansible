# General description #

In this directory there is a playbook for unprovisioning DAG (Distributed Anycast Gateway) for Campus EVPN Fabric.

# Playbooks description #

**playbook_delete.yml:**
- for partial/full deleting DAG related config


# OVERLAY CONFIGURATION #

Overlay configuration is mostly the same on all VTEPs. There are some exceptions like unique interfaces/loopbacks or additonal configuration because of device role (Border) or design (for example CGW). For DAG EVPN/NVE related config is the same on all VTEPs, so it is stored in **group_vars/all.yaml** and applicable to all VTEPs.

Elements which are mentioned in the file **group_vars/all.yaml** with action **delete** will be unconfigured.
It is possible to delete only part of the configuration. Only elements, that are mentioned in the file, will be deleted.
For example, if only vrf is mentioned, then only vrf will be deleted - not evi,svi etc.

Example of **group_vars/all.yaml:**
```
vrfs:
  blue:             <<< overlay vrf to be deleted
    action: delete  <<< delete action

vlans:              <<< vlans associated with the l2vni 
#vrf blue vlans
 201:
  action: delete

 202:
  action: delete

 902:
  action: delete

svis:               <<< svi interfaces associated with the l2vni in the vrf 
#vrf blue svi's
 201:
  action: delete

 202:
  action: delete

 902:
  action: delete

l2vpn_evi:          <<< evi associated with the l2vni 
  201:
    action: delete

  202:
    action: delete

nve_interfaces:    <<< specific dag vnis (l2vni and l3vni) from the overlay topology
  1:
    source_interface: 'Loopback1'
    vni:
      l2vni:
        10201:
          action: delete
        10202:
          action: delete
      l3vni:
        50902:
          action: delete
```
# Full execution 

To run the playbook for deleting the configuration :

```
ansible-playbook -i inventory.yaml playbook_delete.yml
```
