# Mikrotik Configuration Information
#### Revised 25-JAN-2023, John Brown, CISSP
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


## General Information

Before you start you will need to collect some general information and have it available while you work on the configuration.
Some of this information is already in your network configuration, some of it will come from Team Cymru's Client Success Team when you signup.

1. Your ASN (Autonomous System Number):________________<br />
2. The IP on your network that you will be peering from:________________<br />
3. The ASN that Team Cymru provided to you:________________<br />
4. The IP address(s) that Team Cymru provided to you (these may be multiple addresses):________________________<br />

Generally the IP used in item 2 above should be a loopback address on your router.  That way if you are multi-homed that loopback prefix
should continue to be reachable no-matter if you have an outage on one of your provider links.

In the examples below we will use several identifiers (IP's and ASN) to represent the network elements.
These values are for the EXAMPLES ONLY.  You **WILL** need to change them to match the real world.

Customer ASN: 65534<br />
Customer IP Range:  203.0.113.0/24<br />
Loopback:  203.0.113.1/32<br />
Victim IP:  203.0.113.254/32<br />

Team Cymru (these are examples, see your email for real values)<br />
UTRS ASN: 64512 <br />
UTRS-Server-1 IP:  198.51.100.1<br />
UTRS-Server-2 IP:  198.51.100.200<br />

## RouterOS 6.x

The following was tested on a CCR-1009-7G-1C-P<br />
Running RouterOS v6.49.7 on Jan 23,2023

1. We need to create an INPUT filter.  This will take the received UTRS routes and tag them such that the router will _discard_ any traffic TO these routes.  You want to make sure you have an INPUT filter configured before you establish the BGP session with Team Cymru.

`/routing filter
add action=accept append-bgp-communities=no-export,no-advertise bgp-communities=64496:0 chain=tc-utrs-in prefix-length=25-32 set-type=blackhole` <br />
`add action=discard chain=tc-utrs-in`

This filter will match on received routes that have a community string of 64496:0 set.  
For those routes that match they will be installed into the routing table with a type of _blackhole_.   
A blackhole route will cause the Mikrotik to drop traffic going towards that IP address.
We are also only going to match on IPv4 routes that are between a /25 and a /32.  This conforms to the UTRS specification.
Lastly we append two well-known BGP communities, 'No-Export' and 'No-Advertise'   We do this so that these routes do not get advertised to other BGP peers. You don't want to redistrbute these to your IX, Transit or Customer BGP sessions.



2. Next we want to create an OUTPUT filter.  This will only permit YOUR prefixes to be announced to the Team Cymru UTRS BGP Servers.  It is important to have an output filter so that you don't accidentally announce routes that are not your own.
4. Now we will create the BGP Instance for talking to Team Cymru
5. Now we create the BGP PEER's for each Team Cymru UTRS server you will connect to.  We strongly recommend that you connect to at least 2 UTRS servers.  This will provide service redundancy and enhanced resiliancy.
6. Now we will create the BGP NETWORK'S statements.  Prefixes listed here are your **ACTIVE** victims.  Do not list a prefix here unless it is under attack!
7. 


## RouterOS 7.x

The following have been tested on a RB5009UPr+S+IN<br />
Running RouterOS 7.5, build Aug-30-2022


1. We need to create an INPUT filter.  This will take the received UTRS routes and tag them such that your router will discard any traffic TO these routes.
You want to make sure that there is an INPUT filter configured before you establish the BGP session with the UTRS routers

`/routing filter rule
add chain=TC-UTRS-IN disabled=no rule=\
"if(bgp-communities includes 64496:0) {set distance 1; append bgp-communities no-export,no-advertise; set blackhole yes; accept;}"`

We are creating an input rule called TC-UTRS-IN.  The rule will be applied to the UTRS BGP sessions.  This rule will match on BGP routes received from UTRS that have the commmunity string "64496:0" as part of the prefix advertisement.   For routes that match we will ADD the NO-EXPORT and NO-ADVERTISE communities to the prefixes.  We do this to help protect aginst the accidental advertisement out to other BGP neighbors you might have.  We also set BLACKHOLE to YES.  This will cause the Mikrotik to take all packets with a destination to these received routes and drop traffic towards them.

2. Next we will create a TEMPLATE for use with Team Cymru UTRS instances.  

Using a template helps reduce the workload in configuring multiple sessions and makes sure that a consistent policy is applied to all sessions that use this policy.

`/routing/bgp/template
add as=65534 disabled=no input.filter=TC-UTRS-IN multihop=yes name=TC-UTRS-TEMPLATE output.network=TC-UTRS-VICTIM router-id=203.0.113.1 routing-table=main`

This creates a BGP template that sets our local AS to 65534 and also sets the input filter to TC-UTRS-IN.  In addition it sets the output filter
to TC-UTRS-VICTIM.  Any prefix that you put into the TC-UTRS-VICTIM list will be redistributed to the UTRS network for validation and use within the community.

3. Configure the BGP session(s)

Now we are going to configure the actual BGP session(s). We **strongly** recommend that you configure at least two UTRS sessions.  This will provide redundency and higher reliability to the system.

First UTRS Route Server
`/routing bgp connection
add local.role=ebgp name=TC-UTRS-001 remote.address=198.51.100.1/32 .as=64512 .ttl=64 templates=TC-UTRS-TEMPLATE tcp-md5-key=CHANGEMENOW`

Second UTRS BGP Server Configuration

`/routing bgp connection
add local.role=ebgp name=TC-UTRS-002 remote.address=198.51.100.200/32 .as=64512 .ttl=64 templates=TC-UTRS-TEMPLATE tcp-md5-key=CHANGEMENOW`

You will need to change the tcp-md5-key to the key that was assigned by Team Cymru as part of your signup to the service.
Also another **reminder** the IP addresses and ASN's used in these examples need to be changed to the correct values.

At this point you should have a working BGP session with one or more of the UTRS servers.  We *strongly* recommend that you configure at least 2 
BGP sessions.  This will help provide better redudancy and fail-over should one of our UTRS servers goes down.

You can validate that you are receiving routes by using the following command

`/routing/route/print where belongs-to=bgp-IP-198.51.100.1`  
Remember in real life you need to replace the 198.51.100.1 with the real IP for the UTRS router(s) you received from Team Cymru

You can also see the total routes received by issuing the following command

`/routing/stats/origin/print where route-type=8`

Here you will see that the "total-route-count" should be right around 200








