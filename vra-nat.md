---

copyright:
  years: 2017, 2019
lastupdated: "2019-11-14"

keywords: nat, prefix, IPsec, rules

subcollection: virtual-router-appliance

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:note: .note}
{:important: .important}

# Using NAT with prefix-based IPsec
{: #using-nat-with-prefix-based-ipsec}

In [Configuring a VFP interface with IPsec and zone firewalls](/docs/virtual-router-appliance?topic=virtual-router-appliance-configuring-a-vfp-interface-with-ipsec-and-zone-firewalls), a VFP interface was created and set for use with an IPsec tunnel. You can use the same interface in NAT rules, as well as the inbound and outbound interface declaration, with one additional caveat.
{: shortdesc}

Here are some example NAT rules:

```sh
set service nat destination rule 10 destination address '172.16.200.2'
set service nat destination rule 10 inbound-interface 'vfp0'
set service nat destination rule 10 translation address '10.177.137.251'
set service nat source rule 10 outbound-interface 'vfp0'
set service nat source rule 10 source address '10.177.137.251'
set service nat source rule 10 translation address '172.16.200.2'
```

This example is a standard bidirectional one-to-one source and destination NAT for the same IPs. But, to ensure that the NAT traffic goes through the tunnel properly, you need a static route for the other end:

```sh
set protocols static interface-route 172.16.100.2/32 next-hop-interface 'vfp0'
```

The reason for using a static route is because the IPsec daemon already created a kernel route for the remote prefix:

```sh
K    *> 172.16.100.0/24 via 169.63.66.49, dp0bond1
```

Pinging with a source of `10.177.137.251` to `172.16.100.2`, the traffic leaves through `dp0bond1`, fails to transit the tunnel, and never matches the NAT rules properly. The static route fixes this:

```sh
K    *> 172.16.100.0/24 via 169.63.66.49, dp0bond1
S    *> 172.16.100.2/32 [1/0] is directly connected, vfp0
```

This creates a more specific route for the traffic to take through `vfp0`.

At this point, NAT works as configured, and the traffic travels through the tunnel.

NAT requires a route with a CIDR smaller than the IPsec remote prefix (it cannot be the same size) pointing your traffic over the `vfp0` virtual interface.
{: tip}

After everything is in place, you can ping and verify:

```sh
[root@acs-jmat-migserver ~]# ping 172.16.100.1
PING 172.16.100.1 (172.16.100.1) 56(84) bytes of data.
64 bytes from 172.16.100.1: icmp_seq=1 ttl=63 time=44.7 ms
64 bytes from 172.16.100.1: icmp_seq=2 ttl=63 time=44.2 ms
64 bytes from 172.16.100.1: icmp_seq=3 ttl=63 time=44.3 ms
^C
--- 172.16.100.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 44.247/44.431/44.727/0.272 ms

vyatta@acs-jmat-migsim01:~$ show nat source translations
Pre-NAT                 Post-NAT                Prot    Timeout
10.177.137.251:7553     172.16.200.2:7553       icmp    48
```
