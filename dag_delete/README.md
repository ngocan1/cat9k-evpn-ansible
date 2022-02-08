L2/L3VNI w/ Spine Distributed Anycast Gateway (DAG).

Please refer to the following cco guide for the EVPN DAG fabric functionality.

https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/17-6/configuration_guide/vxlan/b_176_bgp_evpn_vxlan_9300_cg/configuring_evpn_vxlan_integrated_routing_and_bridging.html

# playbooks description. #
This example deletes a specific overlay configuration from all Leafs 

```
playbook_delete.yml:
      delete dag overlay config. Please check the group_vars/all.yml

```

# group_vars/all.yml config description. #

**OVERLAY CONFIGURATION**
```

vrfs:
  blue:             <===== overlay vrf to be deleted
    action: delete  <===== delete action

vlans:              <===== vlans associated with the l2vni 
#vrf blue vlans
 201:
  action: delete

 202:
  action: delete

 902:
  action: delete

svis:               <===== svi interfaces associated with the l2vni in the vrf 
#vrf blue svi's
 201:
  action: delete

 202:
  action: delete

 902:
  action: delete

l2vpn_evi:          <===== evi associated with the l2vni 
  201:
    action: delete

  202:
    action: delete

nve_interfaces:    <======= specific dag vnis (l2vni and l3vni) from the overlay topology
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

