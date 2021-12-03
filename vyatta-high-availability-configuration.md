---

copyright:
  years: 1994, 2018
lastupdated: "2018-11-10"

keywords: 

subcollection: virtual-router-appliance

---

{{site.data.keyword.attribute-definition-list}}

# Configuring Vyatta 5400 for High Availability (HA)
{: #vyatta-5400-high-availability-configuration}

Vyatta high availability is supported by using Virtual Routing Redundancy Protocol (VRRP). Each gateway group has two primary VRRP IP addresses, one for the private, and one for the public side of the networks.
{: shortdesc}

Private only Vyattas have only the private VRRP.
{: note}

These IP addresses are the target IPs for the network infrastructure to route all the subnets on VLANs that are associated with the gateway members. Only one Vyatta has these VRRP IPs running at a time. The other members of the gateway group have them administratively down.

Configuration can be synchronized between the two Vyattas with the **config-sync** configuration commands. This configuration allows one member to push the configuration of specific options to the other Vyatta in a group, and do so selectively. You can push only the firewall rules, or only the IPsec configuration, or any combination of rulesets.

It is recommended that you don't push IP addresses or other network configurations, as **config-sync** instantly commits your changes on the other Vyattas, bringing those interfaces online. If you want to dynamically enable interfaces and services, you can use transition scripts to perform this action on failover. Additionally, it is recommended that you use more VRRP IP addresses for your gateway IPs on associated VLANs. This makes failover easier to manage.

Your basic VRRP configuration on both machines appears similar to this sample configuration:

    interfaces {
    bonding bond0 {
    address 10.28.94.213/26
    duplex auto
    hw-id 06:d6:f8:f0:fb:ee
    smp_affinity auto
    speed auto
    vrrp {
    vrrp-group 1 {
    advertise-interval 1
    }
    preempt false
    priority 254
    sync-group vgroup10
    rfc3768-compatibility
    virtual-address 192.168.10.1/24
    }
    }
    }
    bonding bond1 {
    address 50.23.184.5/26
    duplex auto
    hw-id 06:05:09:41:fb:cb
    smp_affinity auto
    speed auto
    vrrp {
    vrrp-group 1 {
    advertise-interval 1
    }
    preempt false
    priority 254
    sync-group vgroup10
    rfc3768-compatibility
    virtual-address 50.23.184.27/26
    }
    }
    }
    loopback lo {
    }
    }

VRRP requires that a real IP address be bound to a virtual interface first before any VRRP advertisements can be sent. In many cases, you can simply add an IP from the primary subnet, but this might conflict with future provisions, or you might want to route a VLAN that already has every primary subnet IP allocated to a server. To get around this, you can use an out-of-band IP address pair on both Vyattas that they can use to talk to each other. Here's an example:

On the first Vyatta:

    set interfaces bonding bond0 vif 1000 address 172.16.0.10/29
    set interfaces bonding bond0 vif 1000 vrrp vrrp-group 10 sync-group vgroup10
    set interfaces bonding bond0 vif 1000 vrrp vrrp-group 10 priority 200
    set interfaces bonding bond0 vif 1000 vrrp vrrp-group 10 virtual-address 192.168.10.1/24

On the second Vyatta:

    set interfaces bonding bond0 vif 1000 address 172.16.0.11/29
    set interfaces bonding bond0 vif 1000 vrrp vrrp-group 10 sync-group vgroup10
    set interfaces bonding bond0 vif 1000 vrrp vrrp-group 10 priority 199
    set interfaces bonding bond0 vif 1000 vrrp vrrp-group 10 virtual-address 192.168.10.1/24

In this case, both Vyattas have their own IP that won't conflict with the subnet that is allocated. You can choose almost any private range you want. Just pick a small subnet that won't conflict with any other routes you might have, such as a subnet range over an IPsec tunnel, or an `10.0.0.0/8` address that conflicts with Softlayer's.

Also add a **sync-group** name. All VRRP addresses should be a part of the same **sync-group**. This is so that any failure on one interface causes all the interfaces in the same **sync-group** to fail over as well. Otherwise, you could end up with some being MASTER, and others in BACKUP. Use the same name in the bond0 and bond1 native VLAN configuration.

The bond0 and bond1 VRRP configuration might have a line for rfc3768-compatibility. This is not needed for VRRP on a VIF, only the native VLAN of bond0 and bond1.
{: tip}

On a newly provisioned gateway pair, **config-sync** only has a minimal configuration:

    config-sync {
    remote-router 10.28.94.195 {
    password ****************
    sync-map syncme
    username vyatta
    }

You must add rules to determine which configuration branches to migrate over. For example, if you want to have the firewall and IPsec configuration to be pushed, add these commands:


    set system config-sync sync-map syncme rule 1 action include
    set system config-sync sync-map syncme rule 1 location firewall
    set system config-sync sync-map syncme rule 2 action include
    set system config-sync sync-map syncme rule 2 location "vpn ipsec"

After these are committed, any changes to the firewall or IPsec configuration are pushed to the other Vyatta on commit.

**sync-group** and **sync-map** are two different things. The **sync-map** configuration is for rules to push configuration changes to another Vyatta. **sync-group** is for VRRP IPs to fail over as a group instead of one at a time. Configuring one does not affect the other.
{: tip}
