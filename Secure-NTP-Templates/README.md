# Secure NTP Template

The Network Time Protocol (NTP) is the de-facto means Internet hosts use to synchronize their clocks.  A reliable and accurate notion of time is important for a number of services, including distributed applications, authentication services, multi-user databases and logging services to name just a few.  The NTP is one of those few systems that sees ubiquitous deployment across systems of all types and sizes.  It is therefore important that the NTP infrastructure is secure and trustworthy.  Negligent NTP configurations can lead to a select set of potential problems, including NTP hosts becoming unwitting participants in reflector and amplification DDoS attacks.  This template provides guidelines for proper and secure operation of the NTP service on a number of different platforms and configurations.

## ! ! W A R N I N G ! !
***
As with all such templates, this one must be modified to fit the specific requirements of the local network(s) and hosts. It is not wise to simply cut and paste without a thorough understanding of each command.
***

Comments are included with each command. A more thorough understanding of NTP can be obtained from:
* [Network Time Protocol (NTP) Project](http://www.ntp.org)
* Computer Network Time Synchronization: The Network Time Protocol, David L. Mills

### Credits
Our thanks to the following folks for providing input and suggestions!
* Alan Amesbury
* Barry Greene
* Anthony Maszeroski
* Donald Smith
* Dnyanesh Savardekar

### General Considerations
All implementations use UDP.  The NTP server port is 123, but the source port is not easily determined without knowledge of the OS and NTP implementation.  We have even seen some implementations use port 123 for both the source and destination port in NTP messages.  While not shown, we strongly encourage the use of IETF BCP 38 to limit spoofed traffic, but particularly to help mitigate spoofed NTP amplification and reflection attacks.  The configurations shown here assume that the host is primarily acting as an NTP client and not an NTP server that delivers time to large populations of anonymous NTP clients.  Those situations are less common and are better suited in another BCP document where additional detail would need to be covered in depth to adequately address time serving issues.
### Cisco IOS

For some recent versions of IOS the NTP access lists that restrict queries, i.e.  query-only, will not take effect. In other words, control mode NTP packets cannot be filtered unless NTP authentication is applied or a fixed version of IOS is  installed. See Cisco bug ID CSCuj66318 and additional discussion on the Cisco  Forums: 
[
https://supportforums.cisco.com/discussion/12061026/ntp-acl-ios-xe-4500-x-bugged](https://supportforums.cisco.com/discussion/12061026/ntp-acl-ios-xe-4500-x-bugged)

This is a template IOS configuration that should work for most sites, but pay attention to the comments and notes.  If your IOS devices synchronize with a device that is capable of MD5 authentication, see further below for authentication specific statements.  If you use control plane policing, be sure you account for NTP traffic.  You might also be interested in adding the `log` tag to some of your ACLs so you know who is trying to talk NTP to your boxes, but that is best left as a local decision so we have not included it by default.


	! Core NTP configuration
	ntp update-calendar             ! update hardware clock (certain hardware only, i.e. 6509s)
	ntp server 192.0.2.1            ! a time server you sync with
	ntp peer   192.0.2.2            ! a time server you sync with and allow to sync to you
	ntp source Loopback0            ! we recommend using a loopback interface for sending NTP messages if possible
	!
	! NTP access control
	ntp access-group query-only 1   ! deny all NTP control queries
	ntp access-group serve 1        ! deny all NTP time and control queries by default
	ntp access-group peer 10        ! permit time sync to configured peer(s)/server(s) only
	ntp access-group serve-only 20  ! permit NTP time sync requests from a select set of clients
	!
	! access control lists (ACLs)
	access-list 1 remark utility ACL to block everything
	access-list 1 deny any
	!
	access-list 10 remark NTP peers/servers we sync to/with
	access-list 10 permit 192.0.2.1
	access-list 10 permit 192.0.2.2
	access-list 10 deny any
	!
	access-list 20 remark Hosts/Networks we allow to get time from us
	access-list 20 permit 192.0.2.0 0.0.0.255
	access-list 20 deny any


Simple NTP authentication using MD5 in IOS can easily be managed for a limited set of static peers and upstream time providers that support it. Since this is generally a manual process, MD5 authentication support for a a large set of clients is likely to be unwieldy.  Nonetheless, this feature provides some additional protection from unwanted NTP messages.  This example assumes that you create an &#8216;ntp authentication-key&#8217; for each peer/server.  The key can be re-used, but we do not recommend re-using the same key with peers or upstreams from different autonomous systems.  Also create a &#8216;ntp trusted-key&#8217; line for each keyid you&#8217;ve configured.  Please note, we have seen some gear limit the pass phrase to eight characters.


	ntp authenticate                            ! enable NTP authentication
	ntp authentication-key [key-id] md5 [hash]  ! define a NTP authentication key
	ntp trusted-key [key-id]                    ! mark a NTP authentication key as trusted
	ntp peer [peer_address] key [key-id]        ! form a authenticated session with a peer
	ntp server [server_address] key [key-id]    ! form a authenticated session with a server


The following commands may prove helpful to monitor or debug NTP issues on IOS.


	! general NTP and clock status
	show ntp status
	! lists synchronization details with configured peer(s)/server(s)
	show ntp associations [detail]
	! shows or logs detailed NTP messages/packets
	! WARNING: not recommended for general use in a production network!!
	debug ntp [...]


### Juniper JUNOS
The following configuration statements will define one or more time servers the router will obtain time from.  The boot server option is used to get a significantly skewed clock back into sync.  To protect the local ntpd process in JUNOS you can use firewall filters on the loopback interface as you likely do for other services.  Authentication can also be done on the Juniper ntpd process and it can be easily managed for a limited set of static peers and upstream time providers that support it.  Since this is generally a manual process, authentication support for a large set of clients is likely to be unwieldy.  Nonetheless, this feature provides some additional protection from unwanted NTP messages.  This example assumes that you create an ntp &#8216;authentication-key&#8217; for each peer/server.  The key can be re-used, but we do not recommend re-using the same key with peers or upstreams from different autonomous systems.  Where you see the [key-id] option, adjust that statement according to your authentication setup, if any.


	system {
	    ntp {
	        authentication-key [key-id] type md5 value "[pass-phrase]";
	        trusted-key [key-id];
	        /* Allow NTP to sync if server clock is significantly different than local clock */
	        boot-server 192.0.2.1;
	        /* NTP server to sync to */
	        server 192.0.2.1;
	        server 192.0.2.2 key [key-id] prefer;
	    }
	}
	

You can use your loopback filter that shields the router from other anonymous access to also limit who the local NTP service talks to.  The relevant section of that filter might look something like the following:


	from {
	    source-address {
	        0.0.0.0/0;
	        /* NTP server to get time from */
	        192.0.2.1/32 except;
	    }
	    protocol udp;
	    port ntp;
	}
	then {
	    discard;
	}


You must then eventually have a default accept-all rule or, if you have a default deny, you must explicitly allow your router to talk to whatever NTP systems it uses for time synchronization.  For example, to add a explicit rule to permit NTP traffic, your configuration might look something like this:


	from {
	    source-address {
	        /* NTP server to get time from */
	        192.0.2.1/32;
	    }
	    protocol udp;
	    port ntp;
	}
	then {
	    accept;
	}


### UNIX ntpd
The following configuration is for a UNIX machine to act as simply an NTP client and never to allow NTP queries to it except from the loopback address:


	# by default act only as a basic NTP client
	restrict -4 default nomodify nopeer noquery notrap
	restrict -6 default nomodify nopeer noquery notrap
	# allow NTP messages from the loopback address, useful for debugging
	restrict 127.0.0.1
	restrict ::1
	# server(s) we time sync to
	server 192.0.2.1
	server 2001:DB8::1
	server time.example.net


You can use your standard host firewall filtering capabilities to limit who the NTP process talks to.  If you&#8217;re using Linux and the host is acting as an NTP client only, the following iptables rules could be adapted to shield your NTP listener from unwanted remote hosts.


	-A INPUT -s 0/0 -d 0/0 -p udp --source-port 123:123 -m state --state ESTABLISHED -j ACCEPT
	-A OUTPUT -s 0/0 -d 0/0 -p udp --destination-port 123:123 -m state --state NEW,ESTABLISHED -j ACCEPT


Authentication with the reference NTP software on UNIX can be done using symmetric key encryption, much like in Cisco IOS and Juniper JUNOS, using MD5.  However, a public key-based approach called &#8216;AutoKey&#8217; is also available, which is generally be considered to be even more secure.  For more information about these options, see the [NTP authentication options page](http://www.eecis.udel.edu/~mills/ntp/html/authopt.html) and the [Configuring Autokey documentation](http://support.ntp.org/bin/view/Support/ConfiguringAutokey).

### A Note About Broadcast/Multicast NTP
In our experience multicast-enabled NTP clients are often setup with no authentication or access control.  Particularly on networks that are connected to the multicast-enabled Internet, these hosts may be exposed to unreliable and untrustworthy NTP servers.  We recommend against multicast NTP configurations unless it is absolutely necessary and even then we strongly suggest the use of access control and authentication to mitigate the threat of untrustworthy sources.  If you do not need multicast NTP support, but you do support IP multicast on your network, you should consider filtering the well known multicast group address for NTP (`224.0.1.1`) at your border.

### A Note About Border NTP Filtering
Some networks may consider filtering all or some NTP traffic between their network and others.  This is potentially very troublesome and should only be considered and implemented with a full understanding of the ramifications.  We cannot advocate this action by default, but can offer some guidelines to those who wish to do so.
All packets to/from TCP port 123 should be safe to filter since NTP by design only uses UDP.  This might, however, affect anyone who attempts to setup another application on TCP port 123 for some reason or any possible future extension of NTP that might use TCP.
Filtering packets from your networks to external networks with UDP source port 123 and/or packets to your networks from external networks with UDP destination port 123 will certainly prevent your hosts from communicating as NTP servers to outside entities, but it may also prevent some NTP hosts from acting as NTP clients as well.  We have seen some clients use port 123 for source ports.  Filtering in this scenario then may cause problems for those clients.
If you can ensure your internal hosts will only act as clients and all legitimate clients will use an unprivileged client port selection strategy you could probably apply the above aforementioned filter. We would recommend logging or monitoring the filters to assist with troubleshooting should it be necessary.  Also, use your systems&#8217;s built in NTP monitoring capabilities to ensure all your NTP client systems remain in sync.
If you block all UDP 123 traffic so that no clients may talk to external servers, you should ensure all your internal hosts are setup to use one or more internal NTP servers.  Since many system components rely on an accurate notation of time and most use NTP to do so, it is important to provide this service.  Note, except for the most limited and restrictive of networks, we do not find it necessary to completely block NTP for all your hosts as long as those hosts can be secured in the kinds of ways suggested in the template configs above.

### References
* [Network Time Protocol (NTP) Project](http://www.ntp.org)
* [Building a cheap NTP stratum-2 network with retired Cisco gear](http://puck.nether.net/pipermail/ednog/2005-June/000048.html)
