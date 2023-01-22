# Mikrotik Configuration Information

## Overview

The following describes how to configure Mikrotik RouterOS version 6.x and 7.x for use with the UTRS service. 
These are just examples.  You should NOT simply cut and paste into your network without fully understanding how they will impact your network
You will more than likely need to modify these to fit how your network operates. 
We strongly recommend that you create a LAB to test the configurations first.

In general using UTRS requires three basic configurations
1. Setting up a BGP peering session with the Team Cymru UTRS route-servers
2. Configuring your router to DROP traffic to IP's that are advertised by the UTRS router
3. Configuring your router to properly send to us a vicitim IP address.

How the above three are accomplised vary depending on if you are running RouterOS 6.x or RouterOS 7.x   The concepts are the same
The actual configuration steps and commands are DIFFERENT.


### RouterOS 6.x


### RouterOS 7.x
