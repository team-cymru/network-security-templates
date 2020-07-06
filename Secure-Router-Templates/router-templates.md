# Secure Cisco IOS BGP Template




### Introduction
BGP is the routing protocol that drives the Internet. Proper configuration of BGP is critical, as mistakes in BGP can result in disaster for both local and remote networks. Further, without a few additional steps to increase the security and defense of BGP, it is possible for miscreants to cause havoc with the BGP and, by extension, routing tables.
This document includes a template configuration for BGP. As with all such templates, this one must be modified to fit the specific requirements of the local network(s). It is not wise to simply cut and paste without a thorough understanding of each command. Comments are included with each command. A more thorough understanding of BGP can be obtained from:

* Internet Routing Architectures, by Bassam Halabi, published by Cisco Press.
* BGP4 by John W. Stewart III, published by Addison-Wesley.

As an aside, debugging BGP issues can be difficult without an external view. To see how the rest of the Internet views your prefix announcements, use the route servers. Simply telnet to one of these route servers and issue commands such as **sh ip bgp NETBLOCK** or **sh ip route NETBLOCK**. Here is a partial list:
* route-views.oregon-ix.net
* route-server.cerf.net
* route-server.ip.att.net
* route-server.east.allstream.com
* route-server.west.allstream.com
* route-server.central.allstream.com
* route-server.cbbtier3.att.net
* route-server.gblx.net
* route-server.opentransit.net
* route-server.gt.ca
* public-route-server.is.co.za (South African routes only)
* route-server.belwue.de
* route-views.on.bb.telus.com
* route-views.ab.bb.telus.com
* route-server.ip.tiscali.net
* route-server.he.net

This great collection of route servers, plus a few more, can be found by querying the following range of DNS RRs:
* routeserver[1-16].sentex.ca

For example:

dig +short routeserver1.sentex.ca
route-views.oregon-ix.net. 198.32.162.100
         - or -         
dig +short routeserver7.sentex.ca  
route-server.gt.ca.         216.18.63.214

Thanks to Mike Tancsa for making this available!
Thomas Kernen maintains an excellent page of route-servers and more at [traceroute.org](http://www.traceroute.org/). Thanks, Thomas!
It may also be helpful to receive the **bgp-stats** report, either daily or weekly. This will help you to size your **maximum-prefix** statements, as well as maintaining accurate bogon filters. You may subscribe to the bgp-stats report by sending a note to [bgp-stats-request@lists.apnic.net](mailto:bgp-stats-request@lists.apnic.net) with the message text of &#8220;**subscribe**&#8220;.
We no longer list bogon blocks statically in the template, instead we suggest taking advantage of our [Bogon Route Server Project](/community-services/bogon reference/bogon-reference-bgp/), part of the [Bogon Reference](/community-services/bogon-reference/). This way your filters will always be current!  You can read more about this at the RIPE NCC [De-Bogonising New Addresses page](http://www.ris.ripe.net/debogon/).
### Credits
Our thanks to the following folks for providing input and suggestions!
* Roy Arends
* Larry Bishop
* Oded Comay
* Piotr Kucharski
* Aristidis Lamprianidis
* Ahmed Maged
* Hank Nussbacher
* Joel Obstfeld
* James A. T. Rice
* Joshua Sahala
* Phillip Smith
* Mike Tancsa
* David Wolsefer
* As always, the [FIRST](http://www.first.org/) community.

### IOS Assumptions
* IOS 12.0.X or higher.
* Understanding of BGP and the Cisco IOS.
* This template is used by a non-transit network.
* The local ASN is 64496, the remote ASNs are 64500 and 64511.
* The local netblock is 192.0.2.0/24.
* The router has already been secured. For details on a secure IOS configuration template, please consult my Secure IOS Template.
* This template was crafted for a network that would be dual-homed and BGP peered to two Tier One ISPs.
* The IP address of the router used in this template is 172.17.70.1.

### IOS Template
The actual commands are in BOLD text so that they stand out from the comment blocks.


! Our ASN is 64496
**router bgp 64496**
!
! Set graceful restart-time to 120 and stalepath-time to 360 for route handling optimization.
! Time listed is in seconds
**bgp graceful-restart**
!
! Don't wait for the IGP to catch up.
**no synchronization**
!
! Be a little more forgiving of an occasional missed keepalive.
**no bgp fast-external-fallover**
!
! Track and punt, via syslog, all interesting observations about our
! neighbors.
**bgp log-neighbor-changes**
!
! Set Maximum AS-Path Prepends to 10 to limit an insane number of prepends.
! The Cisco IOS command, which would limit prepends to a sane level would be :
**bgp maxas-limit 10**
! (supported from 12.2, 12.0(17)S, 12.2(33)SRA, 12.2SX and upwards, see
! http://www.cisco.com/en/US/docs/ios/iproute/command/reference/irp_bgp1.html#wp1013932
! for more details)
!
! Announce our netblock(s) in a manner that does not increase CPU
! utilization. Redistributing from an IGP is dangerous as it increases
! the likelihood of flapping and instability. Redistributing static is
! more stable, but requires the CPU to peruse the routing table at a set
! interval to capture any changes. The network statement, combined with
! a null route, is the least expensive (in terms of CPU utilization) and
! most reliable (in terms of stability) option.
**network 192.0.2.0 mask 255.255.255.0**
!
! Our first neighbor, 10.10.5.1, is an eBGP peer with the ASN of 64511.
**neighbor 10.10.5.1 remote-as 64511**
!
! Cisco provides a TTL security check feature. This is designed
! to limit the number of remote devices that can send BGP (TCP
! 179) packets to our router. Ideally this will lock down BGP
! access to the remote peering router only; it will certainly
! help, especially when coupled with other security techniques
! such as ACLs. Note that this isn't a 100% solution in a LAN
! (e.g. peering exchange) environment. It also must be adjusted
! so that the correct number of hops is set. Please verify the
! hop count using the 'trace' command before setting this option.
! You can read more about this feature here:
! http://www.cisco.com/en/US/docs/ios/12_3t/12_3t7/feature/guide/gt_btsh.html
!
**neighbor 10.10.5.1 ttl-security hops 2**
!
!
! Set for soft reconfiguration, thus preventing a complete withdrawal
! of all announced prefixes when clear ip bgp x.x.x.x is typed.
**neighbor 10.10.5.1 soft-reconfiguration inbound**
!
! Type in a description for future reference. Not everyone memorizes
! ASNs. :-)
**neighbor 10.10.5.1 description eBGP with ISP64511**
!
! Set up a password for authentication.
**neighbor 10.10.5.1 password bgpwith64511**
!
! Hard-set for version 4. Disabled BGP version negotiation, thus
! bringing the peering session on-line more quickly.
**neighbor 10.10.5.1 version 4**
!
! Block any inbound announcments that include bogon networks. A prefix
! list is used because it is:
!  1) Easier on the CPU than ACLs, and
!  2) Easier to modify.
! See the actual bogons prefix-list below.
**neighbor 10.10.5.1 prefix-list bogons in**
!
! Announce only those networks we specifically list. This also prevents
! the network from becoming a transit provider. An added bit of protection
! and good netizenship. See the announce prefix-list below.
**neighbor 10.10.5.1 prefix-list announce out**
!
! Prevent a mistake or mishap by our peer (or someone with whom our peer
! has a peering agreement) from causing router meltdown by filling the
! routing and BGP tables. This is a hard limit. At 75% of this limit,
! the IOS will issue log messages warning that the neighbor is approaching
! the limit. All log messages should be sent to a remote syslog host.
! The warning water mark can be modified by placing a value after the
! maximum prefix value, e.g. maximum-prefix 250000 50. This will set the
! IOS to issue warning messages when the neighbor reaches 50% of the limit.
! Note that this number may need to be adjusted upward in the future to
! account for growth in the Internet routing table.
**neighbor 10.10.5.1 maximum-prefix 250000**
!
! Our next neighbor is 10.10.10.1, an eBGP peer with the ASN of 64500.
**neighbor 10.10.10.1 remote-as 64500
neighbor 10.10.10.1 ttl-security hops 2
neighbor 10.10.10.1 soft-reconfiguration inbound
neighbor 10.10.10.1 description eBGP with ISP64500
neighbor 10.10.10.1 password bgpwith64500
neighbor 10.10.10.1 version 4
neighbor 10.10.10.1 prefix-list bogons in
neighbor 10.10.10.1 prefix-list announce out
neighbor 10.10.10.1 maximum-prefix 350000**
!
! This is our iBGP peer, 172.17.70.2.
**neighbor 172.17.70.2 remote-as 64496
neighbor 172.17.70.2 ttl-security hops 2
neighbor 172.17.70.2 soft-reconfiguration inbound**
!
! Again, a handy description.
**neighbor 172.17.70.2 description iBGP with our other router**
!
**neighbor 172.17.70.2 password bgpwith64496**
! Use the loopback interface for iBGP announcements. This increases the
! stability of iBGP.
**neighbor 172.17.70.2 update-source Loopback0
neighbor 172.17.70.2 version 4
neighbor 172.17.70.2 next-hop-self
neighbor 172.17.70.2 prefix-list bogons in
neighbor 172.17.70.2 maximum-prefix 250000**
!
! Do not automatically summarize our announcements.
**no auto-summary**
!
! If we have multiple links on the same router to the same AS, we like to
! put them to good use. Load balance, per destination, with maximum-paths.
! The limit is six. For our example, we will assume two equal size pipes
! to the same AS.
**maximum-paths 2**
!
! Now add our null route and the loopback/iBGP route. Remember to add
! more specific non-null routes so that the packets travel to their
! intended destination!
**ip route 192.0.2.0 255.255.255.0 Null0
ip route 192.0.2.0 255.255.255.128 192.168.50.5
ip route 192.0.2.128 255.255.255.128 192.168.50.8
ip route 172.17.70.2 255.255.255.255 192.168.50.2**
!
! We protect TCP port 179 (BGP port) from miscreants by limiting
! access. Allow our peers to connect and log all other attempts.
! Remember to apply this ACL to the interfaces of the router or
! add it to existing ACLs.
! Please note that ACL 185 would block ALL traffic as written. This
! is designed to focus only on protecting BGP. You MUST modify ACL
! 185 to fit your environment and approved traffic patterns.
**access-list 185 permit tcp host 10.10.5.1 host 10.10.5.2 eq 179
access-list 185 permit tcp host 10.10.5.1 eq bgp host 10.10.5.2
access-list 185 permit tcp host 10.10.10.1 host 10.10.10.2 eq 179
access-list 185 permit tcp host 10.10.10.1 eq bgp host 10.10.10.2
access-list 185 permit tcp host 172.17.70.2 host 172.17.70.1 eq 179
access-list 185 permit tcp host 172.17.70.2 eq bgp host 172.17.70.1
access-list 185 deny tcp any any eq 179 log-input**
!
! The announce prefix list prevents us from announcing anything beyond
! our aggregated netblock(s).
**ip prefix-list announce description Our allowed routing announcements
ip prefix-list announce seq 5 permit 192.0.2.0/24
ip prefix-list announce seq 10 deny 0.0.0.0/0 le 32**
!
! Team Cymru has removed all static bogon references from this template
! due to the high probability that the application of these bogon filters
! will be a one-time event. Unfortunately many of these templates are
! applied and never re-visited, despite our dire warnings that bogons do
! change.
!
! This doesn't mean bogon filtering can't be accomplished in an automated
! manner. Why not consider peering with our globally distributed bogon
! route-server project? Alternately you can obtain a current and well
! maintained bogon feed from our DNS and RADb services. Read more at the
! link below to learn how!
!
! https://www.team-cymru.org/community-services/bogon-reference/
!
! END


### IOS XR Template
Thanks Joel Obstfeld!
This IOS XS template is not a 1:1 translation of the above template, since some of the commands/functions used on IOS are not enabled by default in IOS-XR.
The order of the configuration IS important, in that policies need to be defined before the BGP process can reference them. If the BGP process is configured first, making reference to a policy which hasn't yet been parsed, it will return an error.


**router bgp &lt;your asn&gt;
address-family ipv4 unicast**
!
**neighbor x.x.x.x**
! TTL security check, as above and:
! http://www.cisco.com/web/about/security/intelligence/CiscoIOSXR.html#72
! Note that in IOS XR TTL security can be enabled for directly-connected
! peering sessions only, not multihop sessions.
**ttl-security
remote-as 65333
ebgp-multihop 255
description &lt;your description&gt;
update-source Loopback999
password clear &lt;your password&gt;
address-family ipv4 unicast
maximum-prefix 100 90
route-policy drop in
route-policy CYMRUBOGONS out
soft-reconfiguration inbound always**
!
!
!
**route-policy drop
 drop
end-policy**
!
**route-policy CYMRUBOGONS
if (community matches-every BOGONS) then
 set next-hop 192.0.2.1
else
 drop
endif
end-policy**
!
**community-set BOGONS
 65333:888
end-set**
!
**router static
 address-family ipv4 unicast
 192.0.2.1/32 Null0**
!
!
! Define prefix-sets and access-lists that will be used later in the template
**prefix-set pfx_announce_permit**
! The announce prefix list prevents us from announcing anything beyond
! our aggregated netblock(s).
** 192.0.2.0/24
end-set**
!
! Team Cymru has removed all static bogon references from this template
! due to the high probability that the application of these bogon filters
! will be a one-time event. Unfortunately many of these templates are
! applied and never re-visited, despite our dire warnings that bogons do
! change.
!
! This doesn't mean bogon filtering can't be accomplished in an automated
! manner. Why not consider peering with our globally distributed bogon
! route-server project? Alternately you can obtain a current and well
! maintained bogon feed from our DNS and RADb services. Read more at the
! link below to learn how!
!
!   https://www.team-cymru.org/community-services/bogon-reference/
!
**prefix-set pfx_bogons_permit**
! Allow all prefixes up to /27. Your mileage may vary,
! so adjust this to fit your specific requirements.
** 0.0.0.0/0 le 27
end-set**
!
! Protect TCP port 179 (BGP port) from miscreants by limiting
! access. Allow peers to connect and log all other attempts.
! Remember to apply this ACL to the interfaces of the router or
! add it to existing ACLs.
! Please note that ACL 185 would block ALL traffic as written. This
! is designed to focus only on protecting BGP. You MUST modify ACL
! 185 to fit your environment and approved traffic patterns.
**ipv4 access-list acl_185
 10 permit tcp host 10.10.5.1 host 10.10.5.2 eq 179
 20 permit tcp host 10.10.5.1 eq bgp host 10.10.5.2
 30 permit tcp host 10.10.10.1 host 10.10.10.2 eq 179
 40 permit tcp host 10.10.10.1 eq bgp host 10.10.10.2
 50 permit tcp host 172.17.70.2 host 172.17.70.1 eq 179
 60 permit tcp host 172.17.70.2 eq bgp host 172.17.70.1
 70 deny tcp any any eq 179 log-input**
!
! Define route-policies to be used by BGP peers
**route-policy announce
 if (destination in pfx_announce_permit) then
 pass
 endif
end-policy**
!
!
**router static**
! Now add our null route and the loopback/iBGP route. Remember to add
! more specific non-null routes so that the packets travel to their
! intended destination!
!
**address-family ipv4 unicast
 192.0.2.0/24 Null0
 192.0.2.0/25 192.168.50.5
 192.0.2.128/25 192.168.50.8
 172.17.70.2/32 192.168.50.2**
!
!
! Now configure BGP peers etc.
! Our ASN is 64496
**router bgp 64496**
!
! Set BGP router-id
**bgp router-id 192.168.1.65**
!
! Be a little more forgiving of an occasional missed keepalive.
**bgp fast-external-fallover disable**
!
! Track and punt, via syslog, all interesting observations about our neighbors.
**bgp log neighbor changes**
!
! IOS-XR has a different feature to accomplish the same limit as
! the 'bgp maxas-limit 10' statement.  You can replace the number
! 10 with any AS hop count you find suitable.
**route-policy Drop_Long_AS-Path
  if as-path length ge 10 then
    drop
  endif
end-policy**
!
! Announce our netblock(s) in a manner that does not increase CPU
! utilization. Redistributing from an IGP is dangerous as it increases
! the likelihood of flapping and instability. Redistributing static is
! more stable, but requires the CPU to peruse the routing table at a set
! interval to capture any changes. The network statement, combined with
! a null route, is the least expensive (in terms of CPU utilization) and
! most reliable (in terms of stability) option.
**address-family ipv4 unicast
network 192.0.2.0/24**
!
! If we have multiple links on the same router to the same AS, we like to
! put them to good use. Load balance, per destination, with maximum- paths.
! The limit is eight. For our example, we will assume two equal size pipes
! to the same AS. maximum-paths ebgp 2
!
! This is our iBGP peer, 172.17.70.2.
**neighbor 172.17.70.2
remote-as 64496**
! Again, a handy description.
**description iBGP with our other router
password bgpwith64496**
! Use the loopback interface for iBGP announcements. This increases the
! stability of iBGP.
**update-source Loopback0
address-family ipv4 unicast
soft-reconfiguration inbound
next-hop-self
maximum-prefix 250000**
!
!
! Our first neighbor, 10.10.5.1, is an eBGP peer with the ASN of 64511.
**neighbor 10.10.5.1
remote-as 64511**
! Type in a description for future reference. Not everyone memorizes
! ASNs. :-)
**description eBGP with ISP64511**
! Set up a password for authentication.
**password bgpwith64511
address-family ipv4 unicast**
! Set for soft reconfiguration, thus preventing a complete withdrawal
! of all announced prefixes when clear ip bgp x.x.x.x is typed.
**soft-reconfiguration inbound**
! Prevent a mistake or mishap by our peer (or someone with whom our peer
! has a peering agreement) from causing router meltdown by filling the
! routing and BGP tables. This is a hard limit. At 75% of this limit,
! the IOS XR will issue log messages warning that the neighbor is approaching
! the limit. All log messages should be sent to a remote syslog host.
! The warning water mark can be modified by placing a value after the
! maximum prefix value, e.g. maximum-prefix 250000 50. This will set the
! IOS XR to issue warning messages when the neighbor reaches 50% of
! the limit.
! Note that this number may need to be adjusted upward in the
! future to account for growth in the Internet routing table.
**maximum-prefix 250000**
! Announce only those networks we specifically list. This also prevents
! the network from becoming a transit provider. An added bit of protection
! and good netizenship. See the announce prefix-list below.
**route-policy announce out**
!
!
! Our next neighbor is 10.10.10.1, an eBGP peer with the ASN of 64500.
**neighbor 10.10.10.1
remote-as 64500
description eBGP with ISP64500
password bgpwith64500
address-family ipv4 unicast
soft-reconfiguration inbound
maximum-prefix 250000
route-policy announce out**
!
! END
