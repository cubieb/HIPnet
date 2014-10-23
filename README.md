CableLabs HIPnet Proof of Concept OpenWRT; ISC DHCP, and the NETGEAR WNDR 3800
===============================================================================

October 15, 2014

Michael Kloberdans, CableLabs Lead Architect/Home Networking
m.kloberdans@cablelabs.com

Brian Otte, CableLabs Software Engineer 
b.otte@cablelabs.com

Aaron Quinto, CableLabs Engineer, Tiger Team 
a.quinto@cablelabs.com


Warnings
--------
* The provided image and code are not suitable for use in a home router.
* This code release is proof of concept only.
* This code demonstrates a subset of CableLabs draft "A Near Term Solution for Home IP Networking (HIPnet);" it does not provide a complete implementation of HIPnet.
* This code supports connecting only one downstream HIPnet router.  If more than one router is connected, then connect only one router and reboot.
* This version of HIPnet supports only given prefix sizes smaller than a /51.
* The provided image and code are not tested.  The code was verified for correctness in a three deep (CER, IR1, IR2) configuration given ISP delegated prefixes /52, /56, and /60.
* Image, code modifications, and router configuration are for the NETGEAR WNDR 3800 only.
* This code is not presented or recommended as architecture, methodology, or best practice.
* Please see the HIPnet.license file in this repository.


About CableLabs verification envirionment
-----------------------------------------
* Cisco CNR, CMTS, Cable Modem.
* CNR provisioning CM, CPE and Prefix Delegation pools.
* CMTS configured for Primary (CM) and Secondary (CPE) IPv4 subnets.
* The CMTS processes CPE IPv6 router advertisements.  For a non-CMTS environment, a router that processes IPv6 router advertisements is required.


About this Repository
---------------------
This repository contains modified ISC DHCP IPv6 Client and Server 'c' code for the purpose of demonstrating proof of concept of a subset of CableLabs draft "A Near Term Solution for Home IP Networking (HIPnet)."  This repository contains a ready-to-flash, HIPnet-enabled, NETGEAR 3800 image.  To generate a HIPnet-enabled NETGEAR 3800 image from source code, use OpenWRT (including the OpenWRT ISC DHCP 'package') to cross-compile this repository's code.  A brief explanation of generating a HIPnet-enabled image is provided below.

This documentation assumes operational familiarity with the ISC DHCP Client and Server, with OpenWRT core, packages, and image generation, and with flashing the NETGEAR 3800.

The provided implementation modifies the IPv6 DHCP client and server to sub-delegate prefixes, and to provision IPv6 and IPv4 routing without the use of a routing protocol.  Please see:

[http://tools.ietf.org/html/draft-grundemann-homenet-hipnet-00] (http://tools.ietf.org/html/draft-grundemann-homenet-hipnet-00)

[http://www.cablelabs.com/the-future-of-home-networking-putting-the-hip-in-hipnet] (http://www.cablelabs.com/the-future-of-home-networking-putting-the-hip-in-hipnet)

This repository also contains a required OpenWRT 'files' directory for the NETGEAR 3800.

This repository is not processing git Pull Requests.


About this proof of concept implementation and code
---------------------------------------------------
The purpose of the code provided is to demonstrate HIPnet proof of concept.  This code is not presented or recommended as architecture, methodology, or best practice.  A good coding methodology for prefix sub-delegation is bit masking.  The provided code uses character array manipulation, illustrating the prefix sub-delegation algorithm.

####Known
* Count of router's VLANs.
* Received prefix delegation.
* HIPnet requirements.

1. Set variables for known values VLAN count, received IANA and IAPD.
2. Determine the prefix sub-delegation configuration mode: depth or width.
3. Determine the bit boundary to use for determining the number of subnets.
4. Determine the number of subnets.
5. Determine the number of subnet bits to use for determining the number of networks.
6. Determine the number of networks.
7. Determine the number of /64 networks in a sub-delegated prefix.
8. Determine the size of the delegated prefixes.
9. Determine the sub-delegated prefix at the top of the address pool.
10. Determine sub-delegated prefix range.
11. Provision v6.
12. Extend prefix sub-delegation to additional configurations (CER, firewall, IPv4.).


HIPnet functionality demonstrated through ISC DHCP code modifications
----------------------------------------------------------------------
* Recursive prefix sub-delegation within the range (inclusive) /52 to a /64;
      HIPnet 'depth' mode only, only one connected downstream router.
* Multiple Address Family Support for IPv4 using Link-ID
* Hierarchical Routing ' IPv6 and IPv4 addressing and routing (default up, specific down routes).
* Edge Detection using the "/48 Check"
* Edge Detection (CER) functionality using firewall settings and IPv4 route provisioning


HIPnet functionality not demonstrated in these code modifications
-----------------------------------------------------------------
* Recursive prefix sub-delegation from a /48 to a /51.
* HIPnet 'width' configuration.
* This code supports one downstream HIPnet IR (internal router).  If more than one router is connected to this router's LAN, then connect only one router and reboot.
* CER-ID.
* Evaluation of multiple DHCP offers (advertisements).
* Multiple ISPs (Backup Connection, Multi-homing, Failover).
* Multicast Support (Service Discovery, Multicast Proxy Support).
* DHCPv4 Server support for Link-ID calculated IPv4 addresses.
* DNS


Provided .c files
-----------------
* dhclient.c
* db.c
* inet.c
* dhc6.c.X (not integrated)


- dhclient.c  (isc-dhcp-ipv6/dhcp-4.2.4/client/dhclient.c)
Generates prefix sub-delegation.  Note this code implements prefixes as arrays of characters that illustrate the prefix sub-delegation algorithm.  Generating prefixes would be better implemented with bit masking.  This code adds the v6 and v4 upstream routes.  This code will not support a 'wide' configuration or more than one connected downstream router.

- db.c  (isc-dhcp-ipv6/dhcp-4.2.4/server/db.c)
Generates the specific v6 and v4 downstream routes (route injection) to the downstream router LAN.

- inet.c  (isc-dhcp-ipv6/dhcp-4.2.4/common/inet.c)
inet.c contains a function, is cidr mask valid(), that is modified such that the DHCP Server runs with the HIPnet generated 'prefix6' declaration.

- dhc6.c.X (not integrated)
Do not build into images until this file has been reviewed and corrected.  dhc6.c.X is provided as a possible path to a solution to compare lease options and return a value indicating which interface is more 'up.'

* The code modifications comprise less than 400 lines of code and comments.


Software versions used in development
-------------------------------------
* OpenWRT version: Barrier Breaker, r41098
* ISC DHCP version: 4.2.4, 29 May 2012


About the NETGEAR WNDR 3800
---------------------------
The NETGEAR 3800 has one dedicated WAN RJ-11 and is a 'directional' router.  UP Detection is not applicable.  By default, the 3800 LAN IPv4 address is 192.168.1.1.  IR LAN v4 and v6 IPs can be determined from connected CPE addresses.  For the provided image and images produced from this code, connect to router LAN via ssh.
        root / cablelabs


To generate a HIPnet-enabled NETGEAR 3800 image using the provided ISC DHCP code
--------------------------------------------------------------------------------
These instructions assume operational familiarity with the ISC DHCP Client and Server, with OpenWRT core, packages, and image generation, and with flashing the NETGEAR 3800.

1. Generate (make menuconfig) an OpenWRT configuration incorporating CONFIG below.
2. Include the ISC DHCP OpenWRT package.
3. Untar provided files.tar to the 'files' directory located in the OpenWRT core base directory.
4. Use make to initially populate the OpenWRT ISC DHCP package default source files.
5. Use the provided .c files to replace the existing ISC DHCP source files.
6. make


OpenWRT configuration, included and excluded packages
-----------------------------------------------------
* CONFIG PACKAGE odhcp6c is not set.
* CONFIG PACKAGE odhcpd is not set.
* CONFIG PACKAGE dnsmasq is not set.
* CONFIG PACKAGE dnsmasq-dhcpv6 is not set.
* CONFIG BUSYBOX DEFAULT UDHCPC6 is not set.
* CONFIG BUSYBOX DEFAULT UDHCPD is not set.
* CONFIG BUSYBOX DEFAULT FEATURE UDHCPC ARPING is not set (Note, this should be set).
* CONFIG BUSYBOX DEFAULT UDHCPC=y.
* CONFIG TARGET ar71xx generic WNDR3700=y.
* CONFIG TARGET BOARD="ar71xx".
* CONFIG TARGET ROOTFS SQUASHFS=y.
* CONFIG PACKAGE busybox=y.
* CONFIG PACKAGE firewall=y.
* CONFIG PACKAGE fstools=y.
* CONFIG PACKAGE kmod-ipv6=y.
* CONFIG PACKAGE microperl=y.
* CONFIG PACKAGE luci-mod-admin-full=y.
* CONFIG PACKAGE ip6tables=y.
* CONFIG PACKAGE iptables=y.
* CONFIG PACKAGE isc-dhcp-client-ipv4=y.
* CONFIG PACKAGE isc-dhcp-client-ipv6=y.
* CONFIG PACKAGE isc-dhcp-omshell-ipv4=y.
* CONFIG PACKAGE isc-dhcp-omshell-ipv6=y.
* CONFIG PACKAGE isc-dhcp-relay-ipv4=y.
* CONFIG PACKAGE isc-dhcp-relay-ipv6=y.
* CONFIG PACKAGE isc-dhcp-server-ipv4=y.
* CONFIG PACKAGE isc-dhcp-server-ipv6=y.
* CONFIG PACKAGE iputils-ping=y.
* CONFIG PACKAGE iputils-ping6=y.
* CONFIG PACKAGE radvd=y.
* CONFIG IB=y.
* CONFIG IPV6=y.
* CONFIG SHADOW PASSWORDS=y.


The 'files' directory
--------------------------
files/etc

    config
        firewall
        network
        radvd
        system
    init.d
        dhclient
        dhclient6
        dhcpd6
        radvd
        [dropbear]
        sysctl.conf
    [shadow]
    [dropbear]

files/usr

    sbin
        dhclient-script

files/root

    scripts for convenient display of router configurations
