Best Current Operational Practice: 
IX Connections
=============
```
Date: 8 October, 2011
Shepard: NEEDED
Subject Matter Expert(s) (SME): Michael K. Smith, Florian Hibler
Status: Draft *(Ratified /possibly different version/ as a BCOP at the IPBCOP session of NANOG 54 on February 6, 2012.)*
BCOP Subject: Public peering exchange participant/member/client configuration

This document is intended to be original content authored by the Global Network Engineering Community 
(GNEC) "at-large" in an organized, democratic, "bottoms-up" approach.
```
  
## BCOP Summary / Appeal
This document describes the physical and network components that comprise the best current operational practices for connecting to a public peering exchange point. It also provides general configuration parameters and guidelines, but does not include vendor-specific configuration information.

## BCOP Background / History
There are a multitude of exchange points around the world each of which has a set of independent providers using a common exchange fabric to connect and pass traffic to one another. Although there are always specific configuration requirements for particular exchange points, there are also common configuration parameters that, if used, will add to the stability of any exchange. This BCOP lays out the guidelines that should apply in most situations, providing a foundation for ￼Exchange-specific requirements, but not superseding them. The specific rules of any exchange must be ￼followed even when contrary to the guidelines specified in the BCOP document.

## BCOP Specifics

This BCOP is divided into four sections:
1) [Physical Layer Guidelines](#section-1-physical-layer-guidelines)
2) [Data Link and Network Layer Guidelines](#section-2-data-link-and-network-layer-guidelines)
3) [Transport Layer Guidelines](#section-3-transport-layer-guidelines)
4) [Other Guidelines](#section-4-other-guidelines)

This BCOP uses the following standard language throughout.
1) Exchange – a common term for an Internet Exchange, Public Peering Point, Exchange fabric, etc.
2) Participant – a common term for any entity or entities connecting into the Exchange.

### Section 1: Physical Layer Guidelines

1) **Direct connection to the Exchange** – when at all possible, a Participant should connect directly from a Layer 3 interface into the shared switch fabric of the Exchange. A direct connection minimizes the likelihood of any extraneous broadcasts or other not allowed control traffic traversing between autonomous Layer 2 environments. Specifically, direct switch-to-switch connections between the Exchange and Participant should be avoided. The potential for impacting the entire fabric and all Participants through spurious Layer 2 events is significantly higher than with a direct connection.

2) **Remote Peering connection to the Exchange** - the Participant is not placing network equipment in a facility enabled by the Exchange (usually not even in the same town or country). Cost savings is the main driver for this configuration. Transport is usually done via MPLS transport (transparent Layer 2 connection) with carriers accredited by the Exchange. Those carriers have to ensure they are compatible with the Exchange’s network (most of the requirements are stated in Section 2 Data Link and Network Layer Guidelines).

	a. Some Exchanges also offer Reseller ports (in combination with Remote Peering). An accredited carrier connects one port to the Exchange and sells multiple Remote Peering connection on this link. Reselling enables Participants to become a member via the carrier at the Exchange without the need of paying for a direct connection to the Exchange.

	*￼Example: If the accredited carrier gets one 10G connection, he is allowed to sell 10 times 1 Gbps, 100 times 100 Mbps or 1000 times 10 Mbps connections to potential Participants. However, the reseller port must not be oversubscribed.*

3) ￼**Confirm transmission characteristics of Layer 1 connectivity** – when connecting to an Exchange for the first time, it is recommended to take a set of baseline readings to use for later comparisons ￼in the event of link issues. Specifically, link integrity should be verified and recorded.

	a. ￼*Fiber Connection* – Power readings on transmit and receive should be taken with an ￼Optical Power Meter (OPM) and recorded for reference. Any questionable readings ￼should be addressed before putting traffic on the link.
	b. *Copper Connection* - Continuity testing should be performed and output recorded for reference. As with fiber, any questionable readings should be addressed before putting traffic on the link.
	c. ￼*Both Fiber and Copper Connections* – once the physical connection has been established, further baseline data should be gathered at the interface. Specifically, a lack of errors (CRC/Framing/Input/Output/Drops, etc.) should be observed and, if not, addressed before putting traffic on the link.

4. **￼Properly label connections** – all connection components should be properly labeled for easy identification using your standard naming conventions. This includes fiber/copper runs, patch ￼panels, interfaces and connected devices.

5. **Create an As-Built of your connection** - a current configuration document will become invaluable in an outage or for knowledge transfer, and should include the following:

	a. *Link by link definition of circuit* - each juncture point on the path between your interface and the Exchange should be recorded, using the connection labels from (3) above.
	b. *Create a contact list for all providers on the Layer 1 chain* - at the very least a contact or list of contacts for the Exchange should be associated with the As-Built and, in the event multiple carriers are involved, contact information should be included for each carrier and associated to their point or points in the Layer 1 path.

### Section 2: Data Link and Network Layer Guidelines

1) **Properly configured IP and subnet mask (both IPv4 and IPv6)** – the appropriate subnet mask must be configured on your connected interface. It is not safe to assume that the mask configuration is ￼similar across different Exchanges.

2) **Ethertypes** - Frames forwarded through an Exchange must only have one of the following types:
	a. 0x0800 – IPv4
	b. 0x0806 – ARP
	c. 0x86dd – IPv6

3) ￼All broadcast and Non-IP protocols must be disabled facing the exchange fabric, or filtered in some way when disabling is not an option. These include, but are not limited to forwarded ￼DHCP, MOP, Ethernet Keepalives, NetBIOS and IPv6 Router Advertisements.

4) ￼All proprietary discovery protocols (CDP, FDP, etc.) must be disabled on exchange-facing interfaces.

5) IP Redirects must be disabled on exchange-facing interfaces.

6) IP Directed-broadcast must be disabled on exchange-facing interfaces.

7) Proxy Protocols - such as Proxy ARP and IPv6 Proxy Neighbor Discovery must be disabled on exchange-facing interfaces.

8) Layer 2 control traffic - in a switch-to-switch configuration, unless otherwise specified, spanning tree must be disabled on exchange-facing interfaces. In the event this is not possible, all control traffic must be suppressed on exchange-facing interfaces.

9) All Interior Gateway Protocols must be disabled on exchange-facing interfaces.

10) Most Exchanges are very restrictive on the amount of MAC addresses on one connection. Usually the limit is 1. If the Participant has any device between the router ports and the Exchange, he should make sure, that this device is completely quiet and not sending any unwanted packets (like stated earlier in this Section). This might lead to hit the MAC security, which is configured on many Exchanges and disables the port, where the Participant is connected, for traffic.

### Section 3: Transport Layer Guidelines

1) Announcing Exchange IPv4/6 subnet via BGP - a participant must not announce the exchange subnets via BGP.

2) Preconfigured Session Turn-UP - a Participant should not preconfigure and enable BGP peering sessions to other Participants until both sides are ready to establish the connection.

3) A Participant should set a maximum prefix of ACL on peers whenever possible.

4) If the Exchange provides route server(s) and the Participant maintains an open peering policy, he should connect to those, to get as much prefixes from other Participants as possible with less effort. Establishing direct BGP sessions between Participants, which exchange mission-critical or lots of traffic, is also important for redundancy.

5) The Participant's NOC should be aware of the Exchange's technical mailing list and be subscribed to it. The subscription address must not be a ticket system with automated replies enabled.

6) If an existing peer of the Participant disconnects from the Exchange, the Participant should remove the session towards that peer immediately to reduce ARP traffic on the Exchange fabric.

### Section 4: Other Guidelines

1) **Security** - there are very few security measures available for connections between Participants and, given the bilateral nature of any interconnection across the Exchange, this BCOP can only describe, but not recommend, any particular method.

	a. MD5 - Providers may use MD5 to secure their BGP session.
	b. MAC Access Control List (ACL) - A provider may use a MAC ACL to ensure remote Participants are establishing BGP sessions from a known source. However, MAC ACLs can be problematic.
	c. IP ACL's - a provider may use IP ACLs to define by IP those participants that are allowed to establish BGP sessions with the Participant's IP address.
	d. Generalized TTL Security Mechanism (GTSM) - Providers should enable GTSM on their peers whenever possible.
	e. Newer BGP security mechanisms are not addressed in this document revision.

2) **PeeringDB** - all Participants should have an updated entry in the PeeringDB.

3) **Routing Registry** - all Participants must have up-to-date objects in the Routing Registry (IRR) of their choice.

  
## BCOP Conclusion / Summary
Be conservative in what you send and liberal in what you receive. Given the sensitive nature of multiple autonomous participants connecting over a shared Layer 2 infrastructure, it is time to consider modifying that adage to “be conservative in what you send and receive.” 

A Layer 2 fabric is certainly the most cost-effective and simple method for allowing multiple systems to connect to one another on a common infrastructure, but it is also the most sensitive to misconfigurations at any point in the chain. It is in the best interest of Exchange Operators and Participants to limit the amount of extraneous traffic hitting the shared fabric in a concerted effort to maintain its stability. 

In a perfect world, we would have only Layer 3 interfaces connected into the shared fabric, and the connected devices would have no unwanted protocols or features enabled. But, there is so much disparity between devices and in many cases, between software revisions and features on a single device, that each one of the guidelines in this BCOP has to be addressed on a connection-by-connection basis.

