# Unwanted Traffic Removal Service (UTRS)

## Introduction

Widespread packet-flooding distributed denial of service (DDoS) attacks continue to inflict harm and damage upon networks and organizations across the globe. Amplification and reflection attacks have only increased the impact of this problem. The Internet has limited native capability to prevent and mitigate these attacks effectively in real-time. These packet flood outbreaks have only partial solutions.

Team Cymru's BGP-based solution is a modern and effective tool that can help mitigate the largest and most concentrated attacks. It can also help eliminate widely agreed upon traffic that is invalid or unwanted. The Unwanted Traffic Removal Service (UTRS) uses BGP to distributes routes and rules to participating networks. We use only vetted information about current and ongoing unwanted traffic from confirmed authoritative sources. Participating in UTRS is open to most networks with registered autonomous system numbers and which originate prefixes into the global Internet routing table. [Note: we are unable to offer this service to other unconnected networks and individuals.]


If you would like to test or participate in this new BGP-based feed
service, please review the technical details in the remainder of this
document and fill out the form on the official Team Cymru 
[UTRS Landing Page](https://www.team-cymru.org/UTRS/).


---


## Table of Contents

1. [ Getting Started ](#getting-started)
2. [ Supported BGP Announcements ](#supported-bgp-announcements)
3. [ Frequently Asked Questions (FAQ) ](#frequently-asked-questions-faq)
4. [ Configuration Guides ](#configuration-guides)
   * [ Cisco IOS ](#cisco-ios)
   * [ Cisco IOS-XR ](#cisco-ios-xr)
   * [ Cisco IOS-XE ](#cisco-ios-xe)
   * [ Mikrotik v7 ](#mikrotik-v7)
   * [ Mikrotik v6 ](#mikrotik-v6)
   * [ Bird2.0 ](#bird20)
   * [ Juniper ](#juniper)
   * Are we missing a template for your gear?  Please reach out to us or submit a pull request.
5. [ Contact Support ](#support)
6. [ Acknowledgments ](#acknowledgments)

---



## Getting Started

All potential UTRS participants will be asked to provide the
following information:

  * Peering AS number
  * Peering IP address (IPv4 or IPv6)
  * UTRS announcement and discusison mailing list subscription email address(es)

Please note the following default attributes of all BGP sessions
and route announcements. Ask about customizing these values to best
suite your local configuration requirements:

  * UTRS local AS number: 64496
  * UTRS local addresses: [provided at provisioning]
  * TCP MD5 password: [provided at provisioning]
  * Next-hop: 192.0.2.1
  * Community: NO_EXPORT
  * Community: 64496:0
  * multihop
  * passive



UTRS peers are expected to drop any and all traffic to any advertised
prefix by configuring the next-hop address to point to a black hole
(.e.g. null0 interface).  See [IETF RFC 3382](https://tools.ietf.org/html/rfc3882)
Configuring BGP to Block Denial-of-Service Attacks for additional information.  

---

## Supported BGP Announcements



IPv4 and IPv6 announcements that adhere to the following guidelines:


* IPv4 prefixes up to a /25.

* IPv6 prefixes up to a /49.



Flow Specification Rules (<i>aka flowspec</i>) that adhere to the following guidelines:

* <u>Must</u> contain a destination address

    * IPv4 prefixes up to a /25.

    * IPv6 prefixes up to a /49.

* <u>Must</u> contain drop action (traffic rate-limit 0).

  * See [ITEF RFC 8955](https://datatracker.ietf.org/doc/html/rfc8955#section-7) for more details.



* <u>May</u> contain a single Source Port, Destination Port, and/or Protocol (Explicit integers only)

Examples of valid rules (in Bird2.0):



* flow4 { dst x.x.x.x/32; proto = 6; dport = 90; sport = 90; } 
* flow4 { dst x.x.x.x/25; proto = 6; dport = 90;  } 
* flow4 { dst x.x.x.x/32; proto = 6;  } 
* flow4 { dst x.x.x.x/32;  } 



## Frequently Asked Questions (FAQ)


**Who is eligible to participate in UTRS?**

 Currently we are making this service available only to operators who
 have an existing ASN assigned and publicly announce one or more
 netblocks with their own originating ASN into the public Internet
 BGP routing table.  We may provide a read-only feed to others in the
 future.



**How much does this service cost?**

 Nothing.  It is a free community service.  We of course would
 appreciate any data sharing contributions or donations that help us
 continue to deliver services like this in the future.  Contact
 [outreach@cymru.com](mailto:outreach@cymru.com?Subject=UTRS-Donations)
 Outreach for additional information.



**Which prefixes does UTRS currently announce?**

 Generally, routing announcements are relatively infrequent. On
 average we will announce a handful of addresses, but it is possible
 we may not announce any routes.  The list may change regularly
 as threats come and go.  We do NOT currently publish the UTRS
 feed for public consumption in real time.



**Will you publish the RTBH routes UTRS announces?**

 RTBH routes that are accepted and relayed by UTRS are also
 announced out-of-band to the UTRS participant community in real
 time, but they are not disseminated publicly.  Routes that do not
 pass verification may also be subject to review.  We may consider
 publishing summary or archival data for public benefit and the
 research community in the future.



**Help I am under attack, can you block it for me?**

 If you represent a network with a registered autonomous system
 (AS) number and can verifiably prove you are the originator of the
 prefix or covering prefix in question, then yes, we may be able to help.
 If we have not worked with you before, however, there may be some
 processing delay before entering your contact information and
 verifying the candidate prefix.



**I don't have an ASN nor use BGP. Can you black hole something for me?**

 We're afraid this is a service only for Internet BGP network
 operators who have been assigned a public use ASN, usually through
 a local or regional Internet registry.  If you need DDoS attack relief
 of the type that UTRS can help provide, you should contact and
 encourage your upstream ISP(s) to talk to us.



**How do you verify peer networks?**

 Sometimes we already have relationships with potential participant
 networks or know people at potential peer network that we already
 trust.  In cases where we do not already know a potential peer, we
 generally obtain and verify UTRS peering requests with one or more
 registered contacts from the regional or local internet registry
 (RIR/LIR) WHOIS databases and the [Peering DB](http://peeringdb.com)
 (so please keep those contact listings up to date).  If we are unable
 to verify contact information using these traditional sources, we can
 usually verify contact information through other public records or
 with peer/upstream networks of the potential participant network.

Additionally If you sign a ROA that permits a prefix to be advertised by another AS, 
we will honor your intention and allow that AS to request a block for addresses within the prefix you have authorized.

**Does UTRS receive route announcements or does it just advertise them?**

 Both, but UTRS does not originate route announcements on its own.  All
 announcements are input from the community via BGP announcements to
 UTRS, which performs verification and relays them in the form of a BGP
 announcement to all other participants.



**Who decides what prefixes are announced to UTRS participants?**

 Each UTRS participant can announce only IPv4 and IPv6 routes for which
 they are authoritatively associated with a covering prefix.  We consider the most specific adverstisment in global BGP tables as authoritative. We also consider ROA's for covering prefixes as authoritative.

**How does UTRS verify route announcements?**

We utilize the [RIPEstat](https://stat.ripe.net/) API to analyze route
 history, origin and AS path.  The peer must be currently the sole
 originator or upstream of the most specific covering route.  The peer
 must also have a history of announcing that covering prefix for at
 least 30 days.  If an announcement does not meet this criteria,
 relaying the route is withheld, and an internal alert may be
 generated and submitted for internal review.



**What other route announcement restrictions are there?**

 A participant is only permitted to have up to 25 active route
 announcements through UTRS at a time.  Additional routes will
 be rejected.  A route announcement also expires after being
 active for seven days whereupon it will automatically be withdrawn.
 A participant can reannounce a route if it wishes to.



**Can I relay my customer black hole routes to UTRS?**

 UTRS presently only verifies prefixes that originate from a direct
 peer, not downstream networks.  This is an often requested feature
 and we are planning to enhance the route verification process to
 be able to support and relay customer AS black hole routes.  A
 participant can request such a route be announced manually by
 contacting the UTRS administration team. A manual review is
 be performed and, if verified, the route is subsequently announced.



**Doesn't UTRS just help "complete" an attack?**

With BGP FlowSpec, you can send things like TCP destination IP 192.0.2.100 and destination port 80 as your block request.  For our peers supporting BGP FlowSpec, they will begin filtering all traffic to TCP 192.0.2.100:80.  This will still permit all other protocols and ports to continue unaltered.  If this server is a mail server not hosting a webmail interface and your attack is to TCP port 80, then you will have successfully filtered the attack while keeping services online. 

 For peers only supporting BGP, Yes. However this decision, made by the victim, may be justified if
 sacrificing a particular target serves the "greater good".  This
 sacrifice may not be a suitable response in some cases.  UTRS is just
 one tool, appropriate in particular circumstances, not all.



**Can we sinkhole traffic instead of black holing it?**

 Yes!  This is a local provisioning decision, but if you prefer to
 provide within your own network a sinkhole for further processing
 and examination, by all means do.  We have also considered offering
 a remote sinkhole service, where you would forward the unwanted
 traffic to a UTRS-supplied sinkhole instance.  If you would be
 interested in relaying unwanted traffic to a remote sinkhole,
 please contact us.



**Can we get flowspec rules instead of just BGP RTBH?**

 Yes, during provisioning time you may request flowspec. If you are already a peer please contact us if you would like to add flowspec support to your session. 





**Can we get your other BGP feeds together with UTRS?**

 Currently UTRS is a separate, standalone service.  We have
 considered the possiblity of offering combined BGP feeds.  If
 this is something that would be helpful to you, please let us
 know so we can adjust our development priorities of the service
 appropriately.



**Can we customize the ASN, next-hop and community tags?**

 Technically yes.  However, we have found few networks need this
 capability and we prefer not to do it to keep our large
 configuration as simple as possible.  You should be able to
 modify and apply any next-hop or community tag with your import
 policy on your own.

**Is a TCP MD5 signature (aka BGP password) necessary?**

 Yes.



**TCP MD5 can cause more problems than it solves, is it really necessary?**

 We require it.  We realize maintaining shared secrets poses
 administrative challenges, and in some cases may reduce performance,
 possibly even leading to certain classes of attacks that may exacerbate
 resource consumption issues.  However, at least with this particular
 BGP service, we feel the protection it does provide is more
 advantageous than any potential disadvantages.



**Must one end of a UTRS BGP session be a passive TCP listener?**

 We kindly request UTRS participants set their end of the BGP peering
 session to passive listener mode.  We feel this option allows us to
 provide the best protection for the UTRS BGP peering traffic while
 being able to more easily run the UTRS BGP server daemons as an
 unprivileged user.  For Cisco routers, this is usually accomplished
 with a statement such as `neighbor <UTRS_1_IP_ADDRESS> transport
 connection-mode passive` and with Juniper a statement such as
 `set protocols bgp neighbor <UTRS_1_IP_ADDRESS> passive`.



**Can I just send UTRS all my normal peering routes?**

 No.  Please do not do that.  That just increases the work our system
 has to perform and we will most likely just ignore them, unless of
 course you send routes which meet our filtering criteria, which we might end up
 passing on to others to black hole.  If we see routes that appear to
 be anything but RTBH routes, we may terminate your peering session.

**What networks are participating in UTRS?**

 We do not publish a complete list of all UTRS participants, because
 not all networks wish to publicly disclose their peering relationships.
 All UTRS participants have the ability to learn who else participates 
 in UTRS along with the ability to interact with other UTRS participants 
 in a community mailing list.  Networks of varying sizes and regions 
 are participating.  


**Anything else you can say about UTRS participants?**

 As of August 23, 2021, we well over 1,200 ASes configured.  Each day, we see
 several different networks advertising routes to UTRS to mitigate
 attack traffic.  We are currently working on some improvements to the
 UTRS system that will make it easier for the world's largest networks
 to participate.


**Is there any other public material about UTRS I can review?**

 * Original announcement of the project given at [Internet Security Days 2014](https://www.cymru.com/jtk/talks/isd2014-utrs.pdf) **create a historic section to house legacy information. 
 * NANOG 62 Lightning Talk: [slides](http://nanog.org/meetings/abstract?id=2436) and [video](https://youtu.be/4yd5fH4j3WM?t=40)
 * As with any good Network Project, there was a NANOG [discussion thread](http://mailman.nanog.org/pipermail/nanog/2014-October/070307.html).
 * CARIS 2015 Lighting Talk: [slides](https://www.iab.org/wp-content/IAB-uploads/2015/04/CARIS_2015_submission_20.pdf)

---

## Configuration Guides

The configuration examples provides are not meant to be copy and pasted into your configurations. These examples include 

* Examples for a redundant BGP session. UTRS is set up in a Active / Active pair, partners can peer to both routers to acieve fault tolerance.
* Examples for sending / accepting IPv4, IPv6, FlowSpec4, and FlowSpec6 routes. Your final configuration may not include all of these examples depending on your network configurations policies.  

---

### Cisco IOS

 This configuration shows examples for IPv4, IPv6, Flowspec4 and Flowspec6 so depending on our network configuration your final configuation may differ.  You
 may also with to apply a strict set of import and export policy
 directives.  Consider this generic template as a guide, not a
 specification.

    ! Do not generate ICMP unreachables or you may run out of CPU
    ip interface Null0
    no ip unreachables
    !
    router bgp 64496
    neighbor <UTRS_1_IP_ADDRESS> remote-as 64496
    neighbor <UTRS_1_IP_ADDRESS> ebgp-multihop 255 
    neighbor <UTRS_2_IP_ADDRESS> remote-as 64496
    neighbor <UTRS_2_IP_ADDRESS> ebgp-multihop 255 
    !
    ! TCP MD5 passphrase provided at provisioning
    neighbor <UTRS_1_IP_ADDRESS> password <UTRS_MD5_PASSWORD>
    neighbor <UTRS_2_IP_ADDRESS> password <UTRS_MD5_PASSWORD>
    !
    ! UTRS will initiate the active open
    neighbor <UTRS1_IP_ADDRESS>  transport connection-mode passive
    neighbor <UTRS2_IP_ADDRESS>  transport connection-mode passive
    !
    ! by default only send RTBH tagged with up to /25 IPv4 /49 IPv6 or FlowSpec that meets our requirments
    neighbor <UTRS1_IP_ADDRESS> route-map utrs-out out
    neighbor <UTRS2_IP_ADDRESS> route-map utrs-out out
    !
    ! ensure you accept the prefixes you are expecting, maybe reject "golden" prefixes
    neighbor <UTRS1_IP_ADDRESS> route-map utrs-in in
    neighbor <UTRS2_IP_ADDRESS> route-map utrs-in in
    !
    ! default UTRS next-hop, if you change it, make sure it forwards to null
    ip route 192.0.2.1 255.255.255.255 null0
    !
    ! Beware of overwriting an existing ACL, or use an existing deny all
    access-list 1 remark utility ACL to deny everything
    access-list 1 deny any
    !
    ! match IPv4 prefixes up to a /25
    ip prefix-list 25-only permit 0.0.0.0/0 ge 25
    !
    !
    ! match IPv6 prefixes up to a /49
    ip prefix-list 49-only perming 0:0:0:0::0/0 ge 49
    !

    ! define RTBH community to tag your UTRS announcements, optional
    ip community-list standard RTBH permit <your-as>:0
    !
    ! permit only announcments meeting the UTRS requirments. 
    ! NOTE: you may wish to whitelist "golden" prefixes
    ! NOTE: consider adding your own community when propagating internally
    route-map utrs-in permit 10
    match ip address prefix-list 25-only
	match ipv6 address prefix-list 49-only
	match ip flowspec dest-pfx prefix-list 25-only
	match ipv6 flowspec dest-pfx prefix-list 49-only
    !
    ! default deny on UTRS updates in case UTRS leaks
    route-map utrs-in deny 100
    match ip address 1
	###nothing similar for IPv6 ??
    !
    ! only send UTRS valid routes RTBH community tag
    route-map utrs-out permit 10
    match ip address prefix-list 25-only
	match ipv6 address prefix-list 49-only
	match ip flowspec dest-pfx prefix-list 25-only
	match ipv6 flowspec dest-pfx prefix-list 49-only
    match community RTBH
    !
    ! do not send UTRS any routes by default
    route-map utrs-out deny 100
    match ip address 1


---

  
### Cisco IOS-XR

 Your configuration may differ slightly, particularly the peering
 attributes that uniquely identify your end of the peering session.  You
 may also with to apply a strict set of import and export policy
 directives.  Consider this generic template as a guide, not a
 specification.
 
 
	! Do not generate ICMP unreachables or you may run out of CPU
	interface Null0
	 ipv4 icmp unreachables disable
	!
	router bgp <your-as>
	 neighbor <UTRS_1_IP_ADDRESS>
	  remote-as 64496
	  ebgp-multihop 255
	  ! TCP MD5 passphrase provided at provisioning
	  password encrypted ##SECRET-DATA
	  ! UTRS will initiate the active open
	  session-open-mode passive-only
	  address-family ipv4 unicast
	   ! ensure you accept the prefixes you are expecting, maybe reject "golden" prefixes
	   route-policy UTRS-in in
	   ! by default only send RTBH tagged with up to /25 IPv4 /49 IPv6 or FlowSpec that meets our requirments
	   route-policy UTRS-out out
	!
	! default UTRS next-hop, if you change it, make sure it forwards to null
	router static
	 address-family ipv4 unicast
	  192.0.2.1/32  Null0
	!
	! define RTBH community to tag your UTRS announcements, optional
	community-set RTBH
	  <your-as>:0
	end-set
	!
	! permit only IPv4 /32 prefixes from UTRS (see prefix-list above)
	! NOTE: you may wish to whitelist "golden" prefixes
	! NOTE: consider adding your own community when propagating internally
	route-policy UTRS-in
	  if destination in (0.0.0.0/0 eq 32) then
	    ! possible direct drop here, no need for static route to Null0
	    ! set next-hop discard
	    done
	  endif
	  drop
	end-policy
	!
	! only send UTRS IPv4 /32's with your RTBH community tag
	route-policy UTRS-out
	  if destination in (0.0.0.0/0 eq 32) and community matches-any RTBH then
	    done
	  endif
	  drop
	end-policy
 
 


---

### Cisco IOS XE 

 Your configuration may differ slightly, particularly the peering
 attributes that uniquely identify your end of the peering session.  You
 may also with to apply a strict set of import and export policy
 directives.  Consider this generic template as a guide, not a
 specification.
 
 
	router bgp 
	 neighbor <UTRS_1_IP_ADDRESS> remote-as 64496
	 neighbor <UTRS_1_IP_ADDRESS> description *** AS64496-V4-TEAMCYMRU-UTRS01 ***
	 neighbor <UTRS_1_IP_ADDRESS> ebgp-multihop 255
	 neighbor <UTRS_1_IP_ADDRESS> transport connection-mode passive
	 neighbor <UTRS_1_IP_ADDRESS> password <MD5_PASSWORD>
	 neighbor <UTRS_1_IP_ADDRESS> update-source Loopback0
	!
	 address-family ipv4
	  neighbor <UTRS_1_IP_ADDRESS> activate
	  neighbor <UTRS_1_IP_ADDRESS> remove-private-as all
	  neighbor <UTRS_1_IP_ADDRESS> soft-reconfiguration inbound
	  neighbor <UTRS_1_IP_ADDRESS> route-map BGP-AS64496-V4-IN-CYMRUUTRS in
	  neighbor <UTRS_1_IP_ADDRESS> route-map BGP-AS64496-V4-OUT-CYMRUUTRS out
	 exit-address-family
	!
	ip community-list standard UTRS permit 64496:0
	ip community-list standard UTRS- permit :0
	!
    ! match IPv4 prefixes up to a /25
    ip prefix-list 25-only permit 0.0.0.0/0 ge 25
    !

    ! match IPv6 prefixes up to a /49
    ip prefix-list 49-only perming 0:0:0:0::0/0 ge 49
	
	ip route 192.0.2.1 255.255.255.255 null0
	!
	route-map BGP-AS64496-V4-OUT-CYMRUUTRS permit 10
	 description IPv4 Filter UTRS learned from cymru.com route-servers
	match ip address prefix-list 25-only
	match ipv6 address prefix-list 49-only
	match ip flowspec dest-pfx prefix-list 25-only
	match ipv6 flowspec dest-pfx prefix-list 49-only
	match community UTRS

	!
	route-map BGP-AS64496-V4-IN-CYMRUUTRS permit 10
	 description IPv4 Filter UTRS learned from cymru.com route-servers
	match ip address prefix-list 25-only
	match ipv6 address prefix-list 49-only
	match ip flowspec dest-pfx prefix-list 25-only
	match ipv6 flowspec dest-pfx prefix-list 49-only
	match community UTRS
	set ip next-hop 192.0.2.1
	!

---

### Juniper

 Your configuration may differ slightly, particularly the peering
 attributes that uniquely identify your end of the peering session.  You
 may also wish to apply a strict set of import and export policy
 directives.  Consider this generic template as a guide, not a
 specification.
 
 
	routing-options {
	    static {
	        # UTRS default next-hop null route
	        route 192.0.2.1/32 {
	            discard;
	            retain;
	        }
	    }
	}
	protocols {
	    bgp {
	        group UTRS {
	            #
	            # enable multihop for remote UTRS peering
	            multihop;
	            #
	            local-as <your-asn>;
	            #
	            # UTRS peering address provided at provisioning
	            neighbor <UTRS_1_IP_ADDRESS> { 
	                description "Team Cymru UTRS";
	                #
	                # allow UTRS to initiate the active open
	                passive;
	                #
	                # import policy, see below
	                import [ UTRS_in DENY_ALL ];
	                #
	                # MD5 passphrase provided at provisioning
	                authentication-key <md5_password>;
	                #
	                # export policy, see below
	                export [ UTRS_out DENY_ALL ];
	                #
	                # do not announce private ASNs
	                remove-private;
	                #
	                # UTRS ASN
	                peer-as 64496;
	            }
	        }
	    }
	}
	policy-options {
	    policy-statement UTRS_out {
	        term BLACKHOLE {
	            from {
	                #
	                # export self-tagged /32 routes only
	                community <your-ibg-blackhole-community>;
	                route-filter 0.0.0.0/0 prefix-length-range /32-/32;
	            }
	            then {
	                #
	                # remove local organization communities before announcing
	                community delete <all-your-ibgp-communties>;
	                #
	                # add UTRS community to trigger blackholing
	                community add UTRS_tag_out;
	                #
	                # add no-export community
	                community add no-export;
	                #
	                # rewrite next-hop
	                next-hop 192.0.2.1;
	                #
	                accept;
	            }
	        }
	    }
	    policy-statement UTRS_in {
	        term BLACKHOLE {
	            from {
	                #
	                # accept tagged /32 routes only
	                community UTRS_tag_in;
	                route-filter 0.0.0.0/0 prefix-length-range /32-/32;
	            }
	            then {
	                # if required, depends on your configuration
	                local-preference 200;
	                #
	                # remove company communities set by 3rd party
	                community delete <all-your-ibgp-communites>;
	                #
	                # add your company community for blackholing within your iBGP
	                community add <your-ibgp-blackhole-community>;
	                #
	                # rewrite next-hop
	                next-hop discard;
	                #
	                accept;
	            }
	        }
	    }
	    policy-statement DENY_ALL {
	        then reject;
	    }
	    community UTRS_tag_in members 64496:0;
	    community UTRS_tag_out members <your-asn>:0;
	    community no-export members no-export;
	}
 
 



### MikroTik V7

 Your configuration may differ slightly, particularly the peering
 attributes that uniquely identify your end of the peering session.  You
 may also wish to apply a strict set of import and export policy
 directives.  Consider this generic template as a guide, not a
 specification.
 
 
	# BGP instance setup
	/routing bgp template
	set default as=<YOUR_ASN> disabled=no router-id=<WAN_IP_ADDRESS> routing-table=main

	# route filters - install these routes as black hole routes
	# do NOT receive or announce anything else by default
	/routing filter rule
	add chain=utrs-in disabled=no rule="if (dst-len in 0-32 && bgp-communities includes 64496:0) { set blackhole yes; accept; }"
	add chain=utrs-in disabled=no rule="reject;"
	add chain=utrs-out disabled=no rule="reject;"

	# UTRS peering session
	/routing bgp connection
	add as=<YOUR_ASN> disabled=no input.filter=utrs-in local.role=ebgp multihop=yes name=UTRS output.filter-chain=utrs-out \
    .no-client-to-client-reflection=no remote.address=<UTRS_IP_ADDRESS> .as=64496 tcp-md5-key=<UTRS_MD5_PASSWORD> router-id=<WAN_IP_ADDRESS> \
    routing-table=main templates=default


 
 
### MikroTik V6

 Your configuration may differ slightly, particularly the peering
 attributes that uniquely identify your end of the peering session.  You
 may also wish to apply a strict set of import and export policy
 directives.  Consider this generic template as a guide, not a
 specification.
 
 
	# BGP instance setup
	/routing bgp instance set default as=<YOUR_ASN> router-id=<WAN_IP_ADDRESS>

	# route filters - install these routes as black hole routes
	# do NOT receive or announce anything else by default
	/routing filter add action=accept bgp-communities=64496:0 chain=utrs-in prefix-length=0-32 comment="" disabled=no invert-match=no set-type=blackhole
	/routing filter add action=discard chain=utrs-in comment="" disabled=no invert-match=no
	/routing filter add action=discard chain=utrs-out comment="" disabled=no invert-match=no

	# UTRS peering session
	/routing bgp peer add address-families=ip disabled=no in-filter=utrs-in instance=default multihop=yes name=UTRS out-filter=utrs-out passive=yes remote-address=<UTRS_IP_ADDRESS>; remote-as=64496 tcp-md5-key=<UTRS_MD5_PASSWORD>


### Bird2.0 

 Your configuration may differ slightly, particularly the peering
 attributes that uniquely identify your end of the peering session.  You
 may also wish to apply a strict set of import and export policy
 directives.  Consider this generic template as a guide, not a
 specification.

    define NODE_IP = <WAN_IP_ADDRESS>;
    define NODE_ASN = <YOUR_ASN>
    define ROUTER_ID = <YOUR_ROUTER_ID>;
    define NEIGHBOR_0_IP = <UTRS1_IP_ADDRESS>;
    define NEIGHBOR_1_IP = <UTRS2_IP_ADDRESS>; 

    define NEIGHBOR_ASN = 64496;
    define NEIGHBOR_PWD = <UTRS_MD5_PASSWORD>

    filter bgp_in_utrs_ipv4
       {
         if  (64496,0) ~ bgp_community && net ~ [ 0.0.0.0/0{25,32} ] then {
             bgp_community.add((<your-ibg-blackhole-community>));
             dest = RTD_BLACKHOLE;
             accept;
        }
           else {
             reject;
           }
       }


    filter bgp_in_utrs_ipv6
       {
         if  (64496,0) ~ bgp_community && net ~ [ 0::/0{49,128} ] then {
             bgp_community.add((<your-ibg-blackhole-community>));
             dest = RTD_BLACKHOLE;
             accept;
        }
           else {
             reject;
           }
       }



    template bgp utrs {
        local NODE_IP as NODE_ASN;
        password NEIGHBOR_PWD;
        multihop;
        passive;
        ipv4 {
                extended next hop;
                import filter bgp_in_utrs_ipv4;
                export all;
        };
        ipv6 {
                extended next hop;
                import bgp_in_utrs_ipv6;
                export all;
        };
        flow4 {
              extended next hop;
              import all;
              export all;
        };

        flow6 {
              extended next hop;
              import all;
              export all;
        };


    }


    protocol bgp utrs1 from utrs {
        description "BGP utrs1";
        neighbor NEIGHBOR_0_IP  as  NEIGHBOR_ASN;

    } 

    protocol bgp utrs2 from utrs {
        description "BGP utrs2;
        neighbor NEIGHBOR_1_IP as NEIGHBOR_ASN;
    } 


---

## Support

 Current UTRS participants may send email support requests to
 [utrsrequest@cymru.com](mailto:utrsrequest@cymru.com?subject=Help).



---
## Acknowledgments

 * Thomas Mangin @ Exa Networks for ExaBGP
 * RIPE NCC for for their RIPEstat service
 * Phillip Baker @ LCHOST (AS25098) for example Cisco IOS configs
 * Theo Voss @ SysEleven (AS25291) for example Juniper JUNOS configs
 * Daniel Suchy @ NFX (AS8251) for example Cisco IOS-XR configs
 * Manish Govindji @ Zee Communications (AS37576) for example MikroTik configs
 * Kamil Mazur @ Skyware (AS42673) for example BIRD configs

Thanks to all those who contribute updates and corrections to this page!
If you would like to be acknowledged publicly for your contributions,
please indicate so when you contact us and we will list your name here. 

##### Updates to: [outreach@cymru.com](mailto:outreach@cymru.com)
##### Or submit a pull request!
