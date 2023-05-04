# Mikrotik Openstack Neutron ML2 Driver

## Why do you need Mikrotik Openstack Neutron ML2 Driver?
Openstack tenant network like vxlan, geneve, gre has significant performance drawback when network card does not support hardware acceleration for vxlan, geneve, gre. In addition, vxlan, geneve, gre will further reducce MTU below 1500 and introduce fragementation, this makes some network sensitive application imposible to run in such openstack deployment environment.

In contrast choose vlan as openstack tenant network simply enable openstack tenant network linerate performance without MTU fragementation. Note: this requires vlan configuration from switch.

If you have Mikrotik switch that runs RouterOS, all you need is a special designed Openstack Neutron ML2 Driver for Mikrotic RouterOS that automatically config vlan for each Openstack tenant vlan network.

## How to make Mikrotik Openstack Neutron ML2 Driver work?

1. Copy mikrotickvlan.py python file to Openstack neutron server driver directory, for dev stack it is /opt/stack/neutron/neutron/plugins/ml2/drivers
2. Find entry_points.txt file on neutron server, for devstack it is under /opt/stack/neutron/neutron.egg-info/entry_points.txt
Find [neutron.ml2.mechanism_drivers] section, add extra line
    ```
    mikrotikvlan = neutron.plugins.ml2.drivers.mikrotikvlan:MikrotikVlanDriver
    ```
3. Open ml2.conf neutron configuration file, depends on Openstack distribution normally under /etc/neutron/plugins/ml2/ml2.conf
Add extra section at end of the ml2.conf file. Here is sample configuration, you need change based on your switch RouterOS configuration.
Assume your openstack compute node A external public network interface connect to a Mikrotick switch running RouterOS, on interface with interface name sfp4. Compute node B external public network interface connect to a Mikrotick switch running RouterOS, on interface with interface name sfp5. Both interface sfp4,sfp5 is slave interface of bridge with bridge interface name 'bridge'. 
    ```
    [mikrotik]
    user = admin
    password = admin
    address = 10.0.50.238
    port = 8728
    use_ssl = false
    create_bridge_vlan_interface = true
    bridge_name = bridge
    bridge_port_interface_name = sfp4,sfp5
    ```
4. Find [ml2] section
Change tenant_network_types to vlan. 
5. Find [ml2_type_vlan] section
Modify Change network_vlan_ranges with a comma seperate list of start end of vlan id you want Openstack tenant network to use in your switch L2 network.
Restart Openstack Neutron service. For devstack 'systemctl restart devstack@q-svc.service'
6. When user create tenant network with a segementation id, /interface/bridge/vlan/ record will be created on vlan-ids equals to segementation id and set tagged bridge port sfp4,sfp5
7.  When user delete tenant network, /interface/bridge/vlan/ record will be delete with vlan-id equal tenant network segementation id.
8. When create_bridge_vlan_interface = true in ml2.conf configratuion file, extra vlan interface created for bridge interface /interface/ for potential L3 routing purpose.

## When you create a Openstack Tenant network
   It auto generate a availabe network segment id, in this case it is 3434
<img width="531" alt="image" src="https://user-images.githubusercontent.com/118003549/236243550-62aa970e-8266-4387-9483-fbd118a687c4.png">

## Mikrotik Openstack Neutron ML2 Driver created following objects on RouterOS
   1. A bridge vlan item tagged with interface sfp4,sfp5
   2. A vlan interface for bridge interface with name vlan3434, vlan-id 3434, this is used for L3 routing if you want enable it later on.
<img width="868" alt="image" src="https://user-images.githubusercontent.com/118003549/236243391-e6afcdab-4443-4882-baea-f7d186fe3ad8.png">

## This driver tested with Mikrotik RouterOS7.8 and suppose to work with RouterOS 7+, for RouterOS 6+ it may not work, please consider upgrade to RouterOS7.
