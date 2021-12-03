---

copyright:
  years: 2017, 2019
lastupdated: "2019-11-14"

keywords: configure, service, NAT

subcollection: virtual-router-appliance

---

{{site.data.keyword.attribute-definition-list}}

# Configuring services
{: #configuring-your-services}
{: help}
{: support}

There are a variety of services running on the {{site.data.keyword.vra_full}} (VRA) that can be configured.
{: shortdesc}

They include:

`vyatta@vrouter# set service`

Possible completions include:

```sh
 > connsync        Connection tracking synchronization (conn-sync) service
 > dhcp-relay      Dynamic Host Configuration Protocol (DHCP) relay agent
 > dhcp-server     Dynamic Host Configuration Protocol (DHCP) for DHCP server
 > dhcpv6-relay    DHCPv6 Relay Agent parameters
 > dhcpv6-server   DHCP for IPv6 (DHCPv6) server
 > dns             Domain Name Server (DNS) parameters
 > flow-monitoring Flow-Monitoring traffic monitoring configuration
 > https           Enable/disable the Web server
 > lldp            LLDP settings
 > nat             Network Address Translation (NAT)
 > netconf         NETCONF (RFC 6241)
 > portmonitor     Portmonitor configuration
 > sflow           sflow configuration for dataplane
 > snmp            Simple Network Management Protocol (SNMP)
 > ssh             Secure Shell (SSH) protocol
 > telnet          Enable/disable Network Virtual Terminal Protocol (TELNET) protocol
 > twamp           Two-Way Active Measurement Protocol
```

`nat` is configured as part of the service, and `https` controls access to the web GUI and the APIs. `connsync` defines how two VRAs can share connection tracking synchronization for things like stateful firewall failover.
