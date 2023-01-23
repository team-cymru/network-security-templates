# Mikrotik Configuration Information

## Overview

The following describes how to configure Mikrotik RouterOS version 6.x and 7.x for use with the UTRS service. 
These are just examples.  You should NOT simply cut and paste into your network without fully understanding how they will impact your network
You will more than likely need to modify these to fit how your network operates. 
We strongly recommend that you create a LAB to test the configurations first.

In general using UTRS requires four basic steps:

1. Signing up for the service via the Team Cymru website.
2. Setting up a BGP peering session with the Team Cymru UTRS route-servers
3. Configuring your router to DROP traffic to IP's that are advertised by the UTRS router
4. Configuring your router to properly send to us a vicitim IP address.

How the above three are accomplised vary depending on if you are running RouterOS 6.x or RouterOS 7.x   The concepts are the same
The actual configuration steps and commands are DIFFERENT.

Plan on about 60 minutes of uninterrupted time to gather information, build the configuration, and test the configuration.

This document will ONLY provide command line interface (CLI) configuration commands.   If you need to use WinBox or WebFig then you will need
to translate the CLI commands into the appropriate menu's on the GUI.


### General Information

Before you start you will need to collect some general information and have it available while you work on the configuration.
Some of this information is already in your network configuration, some of it will come from Team Cymru's Client Success Team when you signup.

1.Your ASN (Autonomous System Number):________________
2. The IP on your network that you will be peering from:________________
3. The ASN that Team Cymru provided to you:________________
4. The IP address(s) that Team Cymru provided to you (these may be multiple addresses):________________________

Generally the IP used in item 2 above should be a loopback address on your router.  That way if you are multi-homed that loopback prefix
should continue to be reachable no-matter if you have an outage on one of your provider links.

In the examples below we will use several identifiers (IP's and ASN) to represent the network elements.
These values are for the EXAMPLES ONLY.  You **WILL** need to change them to match the real world.

Customer ASN: 65534
Customer IP Range:  203.0.113.0/24
Loopback:  203.0.113.1/32
Victim IP:  203.0.113.254/32

Team Cymru (these are examples, see your email for real values)
UTRS ASN: 64512
UTRS-Server-1 IP:  198.51.100.1
UTRS-Server-2 IP:  198.51.100.200

#### RouterOS 6.x


#### RouterOS 7.x

The following have been tested on RouterOS 7.5, build Aug-30-2022


First we create a address list.  This address list will contain VICTIM IP's on *YOUR* network

`/ip firewall address-list
add address=203.0.113.254/32 list=UTRS-VICTIM`


Next we will get the general BGP session up and running.

`/routing bgp template
set default= as=65534 disable=no router-id=203.0.113.1 routing-table=main`

`/routing bgp connection
add as=65534 disable=no local.address=203.0.113.1 .role=ebgp-peer .ttl=64 multihop=yes name=TC-UTRS-001 output.network=UTRS-VICTIM remote.address=198.51.100.1/32 .as=64512 .ttl=64 router-id=203.0.113.1 routing-table=main templates=default`



