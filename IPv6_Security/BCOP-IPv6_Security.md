Best Current Operational Practice: 
IPv6 Security Myths
=============

```
Date: 3 June 2013
Subject: IPv6 security
Shepard: Chris Grundemann
Subject Matter Expert(s) (SME): NEEDED (Chris Grundemann, Scott Hogg, Rob Seastrom)
Status: Draft

The content here within is intended to be original content authored by the Global Network Engineering
Community (GNEC) “at-large” in an organized, democratic, “bottoms-up” approach.
```

## BCOP Summary / Appeal

Even all these years into deployment, some common and recurring misconceptions about security in IPv6 remain. 

## BCOP Background / History

After a presentation on this topic, several members of the GNEC requested more formally documenting these realities as a BCOP.

This is a very early draft that simply copies the presentation text into the BCOP format. 

*Next steps should include adding context at least, and converting to give advice for best practices on how to deal with all of these issues (versus merely pointing them out) at most (i.e. drop the "Myths" part and just offer advice)...*

## BCOP Specifics

### MYTH:  I’M NOT RUNNING IPV6, I DON’T HAVE TO WORRY

### REALITY:

YOUR APPLICATIONS ARE USING IPV6 ALREADY
-Linux, Mac OS X, BSD, and Microsoft Vista/Windows 7 systems all come with IPv6 capability, some even have IPv6 enabled by default (IPv6 preferred)
-They may try to use IPv6 first and then fall-back to IPv4
-If you are not protecting your IPv6 nodes then you have just allowed a huge back-door to exist!

YOUR USERS ARE USING IPV6 ALREADY
[File:Secmyth1.jpg](http://nabcop.org/index.php?title=Special:Upload&wpDestFile=Secmyth1.jpg "File:Secmyth1.jpg")

### MYTH:  IPv6 Has Security Designed In

### REALITY:

IPSEC IS NOT NEW
-IPsec exists for IPv4
-IPsec mandates in IPv6 are no guarantee of security

IPv6 was designed 15-20 years ago

Extension Headers
[File:Secmyth2.jpg](http://nabcop.org/index.php?title=Special:Upload&wpDestFile=Secmyth2.jpg "File:Secmyth2.jpg")
[http://www.cisco.com/en/US/technologies/tk648/tk872/technologies_white_paper0900aecd8054d37d.html](http://www.cisco.com/en/US/technologies/tk648/tk872/technologies_white_paper0900aecd8054d37d.html)

Routing Header Type 0 (RH0) – Source Routing
-Deprecated in  [RFC 5095](http://tools.ietf.org/html/rfc5095):
-_The functionality provided by IPv6's Type 0 Routing Header can be exploited in order to achieve traffic amplification over a remote path for the purposes of generating denial-of-service traffic_

Hop-by-Hop Options Header
-Vulnerable to low bandwidth DOS attacks
-Threat detailed in draft-krishnan-ipv6-hopbyhop

Extension Headers are vulnerable in general
-Large extension headers
-Lots of extension headers
-Invalid extension headers

Rogue Router Advertisements (RAs)
-Can renumber hosts
-Can launch a Man In The Middle attack
-Problem documented in  [RFC 6104](http://tools.ietf.org/html/rfc6104)
-_In this document, we summarise the scenarios in which rogue RAs may be observed and present a list of possible solutions to the problem_

Forged Neighbor Discovery messages

ICMP Redirects – just like IPv4 redirects

Many attacks are above or below IP
-Buffer overflows
-SQL Injection
-Cross-site scripting
-E-mail/SPAM (open relays)

### MYTH: NO IPv6 NAT Means Less Security

### REALITY:

Stateful Firewalls Provide Security
-NAT can actually reduce security

### MYTH:  IPv6 Networks are too Big to Scan

### REALITY:

SLAAC - EUI-64 addresses (well known OUIs)
-Tracking!

DHCPv6 sequential addressing (scan low numbers)

6to4, ISATAP, Teredo (well known addresses)

Manual configured addresses (scan low numbers, vanity addresses)

Exploiting a local node
-ff02::1 - all nodes on the local network segment
-IPv6 Node Information Queries ([RFC 4620](http://tools.ietf.org/html/rfc4620))
-Neighbor discovery
-Leveraging IPv4 (Metasploit Framework “ipv6_neighbor”)
-IPv6 addresses leaked out by application-layer protocols (email)

Privacy Addresses ([RFC 4941](http://tools.ietf.org/html/rfc4941))
-Privacy addresses use MD5 hash on EUI-64 and random number
-Often temporary – rotate addresses
-Frequency varies
-Often paired with dynamic DNS (firewall state updates?)
-Makes filtering, troubleshooting, and forensics difficult
-Alternative: Randomized DHCPv6
-Host: Randomized IIDs
-Server: Short leases, randomized assignments

### MYTH:  IPv6 is too New to be Attacked

### REALITY:

Tools are already available
-THC IPv6 Attack Toolkit
-IPv6 port scan tools
-IPv6 packet forgery tools
-IPv6 DoS tool

Bugs and Vulnerabilities Published
-Vendors
-Open source software
  
Search for “securityfocus.com inurl:bid ipv6”

### MYTH:  96 more bits, no magic (It’s just like IPv4)

### REALITY:

IPv6 Address Format is Drastically new
-128 bits vs. 32 bits
-Hex vs. Decimal
-Colon vs. Period
-Multiple possible formats (zero suppression, zero compression)
-Logging, grep, filters, etc.

Multiple addresses on each host
-Same host appears in logs with different addresses

Syntax changes
-Training!

### MYTH:  Configure IPv6 Filters Same AS IPv4

### REALITY:

DHCPv6 && ND introduce nuance
-Neighbor Discovery uses ICMP
-DHCPv6 message exchange:
-Solicit: [your link local]:546 -> [ff02::1:2]:547
-Advertise: [upstream link local]:547 -> [your link local]:546
-and two more packets, both between your link locals.

  
Example Firewall Filter (mikrotik):
```
Flags: X - disabled, I - invalid, D - dynamic 0 ;;; Not just ping - ND runs over icmp6.
chain=input action=accept protocol=icmpv6 in-interface=ether1-gateway
1 chain=input action=accept connection-state=established in-interface=ether1-gateway
2 ;;; related means stuff like FTP-DATA
chain=input action=accept connection-state=related in-interface=ether1-gateway
3 ;;; for DHCP6 advertisement (second packet, first server response)
chain=input action=accept protocol=udp src-address=fe80::/16 dst-address=fe80::/16 in-interface=ether1-gateway dst-port=546
4 ;;; ssh to this box for management (note non standard port)
chain=input action=accept protocol=tcp dst-address=[myaddr]/128 dst-port=2222
5 chain=input action=drop in-interface=ether1-gateway
```
  

### MYTH:  IT supports IPv6

### REALITY:

It probably doesn’t
-Detailed requirements (RFP)
-RIPE-554
-Lab testing
-Independent/outside verification

### MYTH:  There are no IPv6 Security BCPs yet

### REALITY:

There Are!
-Perform IPv6 filtering at the perimeter
-Use RFC2827 filtering and Unicast RPF checks throughout the network
-Use manual tunnels (with IPsec whenever possible) instead of dynamic tunnels and deny packets for transition techniques not used
-Use common access-network security measures (NAC/802.1X, disable unused switch ports, Ethernet port security, MACSec/TrustSec) because SEND won’t be available any time soon
-Strive to achieve equal protections for IPv6 as with IPv4
-Continue to let vendors know what you expect in terms of IPv6 security features

### MYTH:  There are no IPv6 Security Resources

### REALITY:

There Are!
-IPv6 Security, By Scott Hogg and Eric Vyncke, Cisco Press, 2009
-Guidelines for the Secure Deployment of IPv6 Recommendations of the National Institute of Standards and Technology
-Search engines are your friend!

## BCOP Conclusion / Summary

-Perform IPv6 filtering at the perimeter
-Use RFC2827 filtering and Unicast RPF checks throughout the network
-Use manual tunnels (with IPsec whenever possible) instead of dynamic tunnels and deny packets for transition techniques not used
-Use common access-network security measures (NAC/802.1X, disable unused switch ports, Ethernet port security, MACSec/TrustSec) because SEND won’t be available any time soon
-Strive to achieve equal protections for IPv6 as with IPv4
-Continue to let vendors know what you expect in terms of IPv6 security features
