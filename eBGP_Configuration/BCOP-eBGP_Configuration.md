Best Current Operational Practice: 
eBGP Configuration
=============

```
Date: 30 October 2014
Subject: How-to configure and validate eBGP properly and consistently
Shepard: Bill Armstrong
Subject Matter Expert(s) (SME): Scott Morris, Aaron Weintraub, Raghav Bhargava, Alex Saroyan, Courtney Smith, Mannan Venkatesan, Russell Harrison, Bill Armstrong
Status: Draft

The content here within is intended to be original content authored by the Global Network Engineering
Community (GNEC) “at-large” in an organized, democratic, “bottoms-up” approach.
```

## BCOP Summary / Appeal

This BCOP aims to provide a singular, consistent view of industry standard eBGP interconnection methodologies focusing on pre and post turn-up validation practices and IRR Etiquette with goal of sharing eBGP KNOW-HOW.

## BCOP Background / History

We (the GNEC) have made many mistakes with IPv4 Peering and Transit configurations and operational practices. As the turn-up of E-BGP sessions with peers and transit providers is a part of daily life, there is an opportunity to do things right from the beginning. While the details of these peering and transit relationships can be varied and specific, the technical realities remain largely the same. These technical realities inform the best practices listed here.

*Next Steps: Flesh out text below, then add config examples (in separate files) for various NOS?*

## BCOP Specifics

### What is BGP

Border Gateway protocol (BGP - RFC4271  [[1]](http://www.google.com/url?q=http://tools.ietf.org/html/rfc4271&sa=D&sntz=1&usg=AFQjCNFx8GtOYY1QFW_44WiQCB1EI28Rew)) is the routing protocol used to exchange routing and reachability information between autonomous systems (AS) on the internet. The protocol is often classified as a path vector protocol.

#### Who needs BGP

BGP is relevant to network administrators of large organizations which connect to two or more ISPs, as well as to Internet Service Providers (ISPs) who connect to other network providers and customers. BGP uses two primary modes of information exchanges to communicate with internal and external peers:

1.  Internal bgp (iBGP)
2.  External bgp (eBGP)

#### Internal BGP (iBGP)

When two BGP-enabled devices are in the same autonomous system (AS), the BGP session between them is called an internal BGP session or iBGP session. BGP uses the same message types on iBGP and external BGP (eBGP) sessions , but the rules for when to send each message and how to interpret each message differ slightl[y.](http://www.google.com/url?q=http://bcop.nanog.org/index.php?title=Special:Upload&wpDestFile=EBGPConfigurationBOCP-DRAFTv0_1_01.png&sa=D&sntz=1&usg=AFQjCNG2j9OJsoeApKKEdFnSuc36FS8Oiw)

[File:](http://www.google.com/url?q=http://bcop.nanog.org/index.php?title=Special:Upload&wpDestFile=EBGPConfigurationBOCP-DRAFTv0_1_01.png&sa=D&sntz=1&usg=AFQjCNG2j9OJsoeApKKEdFnSuc36FS8Oiw)[EBGPConfigurationBOCP-DRAFTv0 1 01.png](http://bcop.nanog.org/index.php?title=Special:Upload&wpDestFile=EBGPConfigurationBOCP-DRAFTv0_1_01.png)

In the above figure R1,R2, R3 have iBGP peer sessions with each other. Likewise R4,R5 have iBGP peer sessions between each other. The purpose of iBGP is to provide a means by which eBGP route advertisements can be forwarded throughout the network. In theory, to accomplish this task you could redistribute all of your eBGP routes into an interior gateway protocol (IGP) such as OSPF or ISIS. This however is not recommended in a production environment because of the following reasons:

1.  **Scalability**: Imagine that you are receiving 500,000 eBGP routes in more than one location and you need to influence the per route exit point in your AS. BGP can handle many more routes than IGP protocols. Thus iBGP is required unless all the routes learned will be announced via eBGP.
2.  **Enforce boundaries of trust/control**: What this means is that BGP has many more knobs than IGP's for controlling what you advertise and receive.
3.  **Flexible tools**: BGP communities, BGP extended communities, local-pref etc these make BGP an attractive way to implement custom routing policies within your own autonomous system ( by using iBGP).

One of the main requirements for iBGP is to have a full mesh connectivity between all the routers within the same autonomous system (AS). This full mesh is a logical mesh of TCP connections and not physical connections. Full mesh in iBGP is needed for proper routing information propagation within the autonomous system amongst all the iBGP speakers. For example in the above figure since router's R1, R2, R3 are in the same autonomous system hence they need to be logically connected to each other.

Doing full mesh via physically connecting all the devices in a environment with 100+ routers does not scale well as to connect 100+ routers which are geographically spread across the country with the long haul links would be very expensive. Therefore to solve the full mesh requirement in iBGP there are two recommended solutions:

1.  Route reflectors [1]
2.  Confederations [2]

[1] Route Reflectors are indicated by a cluster-id. As your network grows, there can be a hierarchy of clusters or route-reflectors in order to enhance the scalability and resiliency of the BGP network.
[2] Confederations bring in particular exceptions to the forwarding rules and blur the lines of typical “internal” versus “external” peers. This may increase the complexity of your network and is recommended to be done if you have extensive BGP knowledge.

#### External BGP (eBGP)

When two BGP-enabled devices are in different autonomous system (AS), the BGP session between them is called an external BGP session or eBGP session. eBGP runs between routers in different autonomous systems. By default, in eBGP (peering in two different AS), IP TTL is set to 1, which means peers are assumed to be directly connected. iBGP does not have this restriction.

In this case, when packet crosses one router, TTL becomes 0 and then the packet will be dropped beyond that. In cases where the two neighbors are not directly connected, for example, peering with loopback interfaces or peering when devices are multiple hops away, we need to add the keyword "Multi-hop" with a TTL value otherwise, BGP neighborship will not be established. In addition, eBGP peer will advertise all the best routes it knows or it has learned from its peers (whether eBGP peer or iBGP peer), which is not, in the case of iBGP.

In the above figure Routers R5, R6 have eBGP sessions with routers R2 & R3 respectively.

#### Route Advertisement in iBGP vs eBGP

##### iBGP

An iBGP peer does not advertise the prefix it learned from an iBGP peer to another iBGP peer. This restriction is there to avoid loops within the same AS. To clarify this, when a route is passed to a eBGP peer, the local AS number gets added to the prefix in AS-path, so if we receive the same packet back stating our AS in AS-path, we know that it is a loop, and that packet gets dropped. However, when a route is advertised to an iBGP peer, the local AS number is not added to AS-path, since the peers are in same AS.

Another important thing to note is that iBGP peers do not change next-hop information when sending updates to other iBGP peers. Unless you are carrying edge link information in your IGP, it is often necessary to use a “next-hop-self” policy between iBGP routers.

##### eBGP

An eBGP peer advertises the prefix it learns from any peer to all other peers i.e internal/external.

#### Loop avoidance mechanism in iBGP vs eBGP

##### iBGP

1.  **Routing Information Loops**: In the case of iBGP session, routing information loops within the AS are prevented by iBGP split-horizon: routing information that is received through an iBGP session is never forwarded to another iBGP neighbor, only toward eBGP neighbors.Because of BGP split-horizon, no router can relay iBGP information within the AS: all routers must be directly updated from the border router that received the eBGP update.
2.  **Attributes** like Originator_id & Cluster_list are attributes which are used to prevent routing loops in iBGP.

##### eBGP

AS-path attribute is used to prevent loops in eBGP sessions. If an eBGP router sees its own autonomous system in the prefix learned via another eBGP peer in the AS-path then it drops that prefix based on the AS-path.

#### BGP best Path selection refresher

[**Note**]: Since there are so many vendors these days we will cover the two most prominently used in the industry (Juniper & Cisco ).

Border Gateway Protocol (BGP) routers typically receive multiple paths to the same destination. The BGP best path algorithm decides which is the best path to install in the IP routing table and to use for traffic forwarding.

##### Juniper

1.  Next-hop accessibility—If the Next-hop is inaccessible, the local router does not consider the route. The router must verify that it has a route to the BGP Next-hop address. If a local route to the Next-hop does not exist, the local route does not include the router in its forwarding table. If such a route exists, route selection continues.
2.  Highest local-preference—The local router selects the route with the highest local-preference value. If multiple routes have the same preference, route selection continues.
3.  Shortest AS-path—The local router selects the route with the fewest entries in the AS-path. If multiple routes have the same AS-path length, route selection continues.
4.  Lowest origin—The local router selects the route with the lowest origin value. If multiple routes have the same origin value, route selection continues.
5.  Lowest MED value—The local router selects the route with the lowest multiple exit discriminator (MED) value, comparing the routes from the same AS only. If multiple routes have the same MED value, route selection continues.
6.  Strictly external paths—The local router prefers strictly external (eBGP) paths over external paths learned through interior sessions (iBGP). If multiple routes have the same strictly external paths, route selection continues.
7.  Lowest IGP route metric— The local router selects the path for which the Next-hop is resolved through the IGP route with the lowest metric. If multiple routes have the same IGP route metric, route selection continues.
8.  Maximum IGP next hops—The local router selects the path for which the BGP Next-hop is resolved through the IGP route with the largest number of next hops. If multiple routes have the same number of next hops, route selection continues.
9.  Shortest route reflection cluster list—The local router selects the path with the shortest route reflection cluster list. Routes without a cluster list are considered to have a cluster list of length 0. If multiple routes have the same route reflection cluster list, route selection continues.
10.  Lowest router ID—The local router selects the route with the lowest IP address value for the BGP router ID. By default, the router IDs of routes received from different ASs are not compared. You can change this default behavior.
11.  Lowest peer IP address—The local router selects the path that was learned from the neighbor with the lowest peer IP address.

##### Cisco

1.  Prefer the path with the highest Weight.
    1.  Note: Weight is a Cisco-specific parameter. It is local to the router on which it is
2.  Prefer the path with the highest LOCAL_PREF configured.
3.  Prefer the path that was locally originated via a network or aggregate BGP subcommand or through redistribution from an IGP.
4.  Prefer the path with the shortest AS_PATH.
    1.  This step is skipped if you have configured the bgp bestpath AS-path ignore command.
    2.  The AS_CONFED_SEQUENCE and AS_CONFED_SET are not included in the AS_PATH
5.  Prefer the path with the lowest origin type.
    1.  IGP > EGP > ?
6.  Prefer the path with the lowest Multi-exit discriminator (MED).
    1.  This comparison only occurs if the first (the neighboring) AS is the same in the two
    2.  In other words, MEDs are compared only if the first AS in the length paths. Any confederation sub-ASs are ignored. AS_SEQUENCE is the same for multiple paths. Any preceding AS_CONFED_SEQUENCE is ignored.
    3.  If bgp always-compare-med is enabled, MEDs are compared for all paths.
    4.  You must disable this option over the entire AS. Otherwise, routing loops can
    5.  If bgp bestpath med-confed is enabled, MEDs are compared for all paths that
    6.  These paths originated within the local confederation.
    7.  THE MED of paths that are received from a neighbor with a MED of occur consist only of AS_CONFED_SEQUENCE.
    8.  THE MED of paths that are received from a neighbor with a MED of
    9.  Paths received with no MED are assigned a MED of 0, unless you have
    10.  If you have enabled bgp bestpath med missing-as-worst the paths are assigned a MED of
    11.  If you have enabled bgp bestpath med missing-as-worst the paths are assigned a MED
    12.  The bgp deterministic-med command can also influence this step.
7.  Prefer eBGP over iBGP paths.
8.  Prefer the path with the lowest IGP metric to the BGP next hop.
9.  Determine if multiple paths require installation in the routing table for bgp multipath.
10.  When both paths are external, prefer the path that was received first (the oldest one). This particular decision factor was introduced by Cisco later on in the evolution of BGP and there is a knob to bypass this step. By keeping the oldest path the best one, route changing across the network is minimized. This is a logical and good goal. However, there are some operational downsides to this mode of operation. If you have 2 transit providers, A and B and some end destination D with all other attributes the same (lowest IGP metric, AS_PATH, etc) then once your system starts up for the first time, it will choose the first path that comes up. Let us say it will be via provider A. If your session to provider A goes down, then your network will prefer provider B. From that point on no matter how much the session to A resets, it will stay on provider B. However, if provider B resets, the traffic will go back to A. In other words, where the traffic is actually going is influenced by the state of the network at that time. If there is a significant amount of traffic going to end destination D, then at any point in time it is essentially random where the traffic to D will go and this could cause issues with traffic management. Configuring this knob to bypass this step takes us to step 11 which makes the decision as to where the route will go is the same at any given time when the external conditions are equal
11.  Prefer the route that comes from the BGP router with the lowest router ID.
12.  If the originator or router ID is the same for multiple paths, prefer the path with the minimum cluster list length.
13.  Prefer the path that comes from the lowest neighbor address.

### Pre-turn-up considerations

Relationship Type

1.  Peering
2.  Transit
3.  Hybrid

Physical Layer Guidelines

1.  Confirm the Media type(Copper, fiber)
2.  Confirm Link Speed

Network Layer Guidelines

1.  Identify Neighbor relationship type(Point to Point, Multi-hop)
2.  Confirm Peering Interface(s) Prefix length and Address-family
3.  Know that this is an edge interface, ensure that Broadcast, Discovery Protocols, IGPs, Proxy protocols are disabled.
4.  Confirm Layer 3 reachability to the remote peer

Transport Layer Guidelines

1.  Identify the anticipated number of prefixes intended to be announced
2.  Confirm the number of prefixes that are expected to be learned by the remote peer.
3.  Identify potential prefix filtering policies which may be applicable to the BGP session.

#### Relationship Types

##### Transit

The most typical form of Internet attachment is Transit. IP Transit is the service of allowing network traffic to cross or “transit” through an intermediate network to reach the larger Internet. From a Provider standpoint this may hold dual meaning, covering those Users which attach to the provider network and purchase transit services as well as those scenarios where BGP is used to exchange routes with a Tier 1 provider to reach the larger Internet.

##### Peering

Peering is a voluntary interconnection of administratively separate Internet networks (Autonomous Systems) for the purpose of exchanging traffic between the customers (or end-users) of each network. With BGP Peering connections an organization will advertise only it's own Prefixes and/or customer routes to peers, these peers will typically do the same in return. In short a ‘peer’ network does not provide connectivity to the Internet. Peering is typically “settlement-free”, meaning neither parties pay the other for the cost of exchanging traffic. If there are costs incurred due to the exchanging of traffic, such costs are typically shared by both parties

#### Interconnection Types

In order to establish a BGP session with another autonomous system, neighboring nodes must be accessible to each other at the network layer. This accessibility can be achieved through a number of different modes of Layer 3 interconnection. Although an obvious requirement, knowledge of what both parties are expecting to meet each other with is crucial.

##### Point to Point Interface peering

Perhaps the most common means of BGP neighbor connectivity is through the use of a single layer 3 interface. The only requirement for this mode of attachment is that a shared layer 2 path and Layer 3 network be used to establish the relationship. In most cases the term Point to Point is indicative of the size prefix expected to be used for the link itself (a /30 IPv4 (or /31) and /126 (or /64) IPv6 address). Follow your current best practices on link addressing.

##### eBGP Multi-hop peering

Default implementations of BGP use a TTL(time-to-live) of 1 and as such will only be capable of establishing a peering relationship with a BGP speaking node on the same layer 3 segment. Multi-Hop peering is enabled with through the use of a remote(not-directly attached) address and thus require a TTL greater than 1. BGP Multi-hop will be leveraged in scenarios where multiple Layer 3 links are used between nodes to increase bandwidth or reliability of the Peering session or in scenarios where some need to abstraction is required by the connecting party. In this scenario the prefix utilized will undoubtedly be a host address (/32 IPv4 or /128 IPv6)

##### Load sharing

When turning up multiple links between networks for eBGP sessions, either Equal Cost Multiple Path (ECMP) or Ether-Bundle (LAG) configuration can be used. Both these options have pros and cons. In ECMP mode, all links that are terminated on the same nodes on both sides are configured as P2P Layer3 link with a /30 IPv4 and /126 (or /64) IPv6 addresses. In LAG bundle mode, all links terminated on same nodes are bundled and configured as a single Layer3 link. Traffic load sharing is done at Layer2 for bundle config and it is done at Layer3 for ECMP config. Layer2/Layer3 load sharing capabilities of the routers that are supporting the peer links should be considered to select ECMP or Bundle configuration.

Certain old router platforms support bundles with 2, 4, 8, 16, 32 links combinations and there is a maximum number of links supported per bundle limitation as well. Bundle configuration offers 'min-link' function to take entire bundle down in partial failure scenarios. This feature is useful when dual bundles are used for redundancy.

### Policy Considerations

#### Inbound policy classification approach.

It is essential to control incoming BGP updates from all the neighbors. In order to maintain scalability it is useful to divide neighbors into groups, In many cases it is possible to group policies as well. There are common and individual policies for each group of BGP neighbors.

Although there countless potential configurations nearly all neighbor types can be classified into 3 common types: Transit, Peers, Downstreams.

**Transits:**  Expect to receive full view but filter "bad" prefixes and allow the rest, assign default (lowest) local-preference, for monetary purposes Transit should be used as an exit point of last resort. From a basic CE standpoint only default route may be learned from transit - in the CE case basic policy should be to allow default route from transit and filter ALL other routes.

**Peers:**  Unlike the transit case, only defined prefixes should be learned and preference should be applied to the paths learned from Peers over paths learned from transits. The goal of We should synchronize policies with IRR database of RPSL or if not possible and just few peers at least configure a manual prefix list, idea is to permit only agreed prefixes. We set local-preference higher than for transits but lower than for Downstreams. Additionally we should configure limit on number of prefixes received.

**Downstreams:**  These are usually customers, so we expect to get some exact prefixes originated in their subnet and prefixes of their customers if they have such, we also want to consider them as a most preferred exit point for the traffic destined to prefixes received from Downstreams. We configure synchronization with IRR database RPSL if not possible and just few Downstreams at least configure manual prefix list to allow only the prefixes which we agreed with each our Downstream, additionally we configure highest local-preference.

- There could be situations when we need to tune some best path selection, For example we have best path for particular prefix over Peer but we want to make it best over transit, or we have best path for particular prefix over one transit but we want to direct outgoing traffic to that prefix over another transit. For such cases it is recommended to keep special clauses within neighbor policy where you can increase or decrease local-preference depending on some prefix list or AS-path list or regexp. This is most-often done through the use of BGP Communities and applicable policy or route-map (depending on vendor).

#### Inbound policy definitions and examples.

Transit providers will send ALL routes thus it is not recommended to define an explicit list of all prefixes for inbound filtering, the goal is to filter all potentially "bad" prefixes and permit the rest.

##### Transit inbound filters both for IPv4 and IPv6

Deny prefixes which AS-path includes AS number from private AS number space

ASN 64512 - 65534, 4200000000 - 4294967294 (RFC6996)

##### Transit inbound IPv4 filters

Deny default route
0.0.0.0/0.

Deny prefixes longer than /24.

Deny prefixes from special use IPv4 address space:
0.0.0.0/8
10.0.0.0/8
127.0.0.0/8
169.254.0.0/16
172.16.0.0/12
192.0.0.0/24
192.0.2.0/24
192.88.99.0/24
192.168.0.0/16
198.18.0.0/15
198.51.100.0/24
203.0.113.0/24
224.0.0.0/4
240.0.0.0/4 (RFC5735)

Deny prefixes from shared address space
100.64.0.0/10 (RFC6598).

If your ASN is one that has downstream BGP customers - i.e. you set up BGP sessions with your customers, you have to allow for the possibility that those customers will purchase additional IP connectivity from other ASNs. Once you have made the step to become a transit AS instead of an end AS, you must allow your own IP space to be announced back to you. This is because either as a planned configuration (your customer cancels and you have allowed them a grace period to renumber) or an unplanned configuration ( there is an outage between you and your customer, causing the BGP between you and them to go down) you would then have to route out to your transit provider to reach that customer.

Since a customer can change from static to BGP and back at any time, it is not operationally feasible to maintain separate address blocks for static routed customers vs BGP routed customers in order to keep your filter list manageable. Instead, simply allow your entire customer address range in from your transit providers.

Deny prefixes from own address space.

Allow the rest

#### Transit inbound IPv6 filters

Deny prefixes from documentation-only IPv6 address space
2001:db8::/32 (RFC3849)

Permit IANA allocated IPv6 scopes
2000::/5 le 48
2800::/6 le 48
2C00::/8 le 48

Deny ALL other IANA reserved IPv6 scopes. deny ::/0 le 128

#### Communities

Communities are used to control some of BGP decision process logic. It is individual from network to network which kind of communities to configure however still it is possible to define some common recommendations.

##### Downstream Communities

Customer communities are communities that a Provider may accept to allow customers to change local-pref, increase their AS-path by pre-pending, control which routes are advertised to peers and transit providers, and black hole traffic at the Provider's edge to reduce impact of DDOS attacks.

**Blackhole:**  Used to instruct BGP neighbor to drop packets destined for particular prefix which is under attack, usually ASN:666 or ASN:999 is used. Router should install a discard/null route for the prefix which has blackhole community and instead of forwarding traffic destined to this prefix just drop the packets. It is also useful to start sending appropriate(because different AS use different community numbers) blackhole communities to neighbors in order to tell them again drop packets destined to this prefix instead of forwarding them. This policy should accept prefixes as long as /32 because in many cases an attack has a vector to a single host and it is worth to blackhole single host and not subnet with more hosts.

**Outbound tweaking:**  Used to let users influence announcements which we send to our neighbors, typically to inform a provider to stop announcing or to prepend particular prefix to particular neighbor or to group of neighbors.

As an example Format of this community consists of two portions one portion is associated with neighbor or with group of neighbors and another portion tells what to do. Final format is up to each operator, here I use numbers just for example. I use first 4 digits for neighbor or group and last 5-th digit to encode the action.

ASN:xxxxy - xxxx will be assotiated with neighbor or group, probably I will code here all my transits, all my major peers, and will make few groups like all transits, all peers, and maybe some groups by country.

ASN:xxxxy - y will keep information about action to be done

0 - Do not announce
1 - Prepend once
2 - Prepend twice
3 - Prepend 3 times
4 - Prepend 4 times

It is completely up to service provider which actions to automate in the BGP router by encoding different commands into communities.

**Inbound marking:**  Another recommended usage of communities is to define community numbers associated with each neighbor and set appropriate community to all accepted prefixes from particular neighbor. This is helpful to identify from which neighbor was the prefix entered into the network.

##### Transit Communities

Transit communities are communities that a provider may add to routes before advertising to transit providers in an effort to dictate where traffic enters their network. Most providers have communities setup to allow customers to change Local-Pref and AS-path length.

A fairly exhaustive list of Transit providers and their associated Customer community model can be referenced here  [[2]](http://www.onesc.net/communities/)

When applying your policies to add communities, be sure they are ‘add’ and not ‘set’. BGP updates can contain multiple communities and it is good neighborship to not destroy other’s communities when applying your own.

#### Outbound policy Considerations

***NEEDS WORK*** If you have downstream BGP customers and you want to re-announce their prefixes to peers, transits, or other customers, on your outbound policy, you should match customers routes with BGP communities (which should have been set on the inbound policy on the BGP session with the customer where you learned that route), instead of doing so with just prefix-lists.

### IRR and PeeringDB

The Internet Routing Registry(IRR) are distributed databases used to publish information about your network. This information is described using Routing Policy Specification Language(RPSL) objects. Service providers commonly use the IRR for automating customer filter generation. There are several entities operating IRR databases. A list of well known databases and more detailed information about RPSL can be found at  [http://irr.net](http://irr.net/).

#### IRR basic building blocks

The following are the basic objects we should maintain in the IRR to allow your peers to generate prefix filters. More complex use cases are outside the scope of this document.

##### Maintainer Object

The maintainer object is used to provide your point of contact information and authorization of object creation and modification. This is a required object.

##### Route/Route6 Object

A route object is used to describe the IPv4 prefixes originated by our ASN. An object should exist for each of our prefix allocations. More specifics are normally not required since many service providers use or-longer in their filters. A route6 object is the IPv6 equivalent.

##### AS-set object

A as-set object is used to reference a group of autonomous system numbers(ASN's) or as-sets. This object is required if we provide or plan to provide transit to other ASN's.

#### Choose a IRR database

When choosing an IRR registry one should consider the following in no particular order:
-   Cost
-   Security options for making updates
-   Is it mirrored by other IRR databases?
-   Can your transit provider use it to generate filters?

#### Set up your IRR

1. **Gather information about our network**
	-   Identify all prefixes allocated to you by your RIR and/or service provider.
	-   Obtain the IRR objects maintained by ASN's you will provide transit.
2. **Create maintainer object** - We must create a maintainer object in our chosen IRR database. Each IRR operator provides their own method for creating the maintainer object.
3. **Create route/route6 objects** - We should create route objects for each prefix allocated to us by our RIR and any allocations /24 or shorter assigned to us by our service provider. The objects should have our ASN as the origin.
4. **Create a_n_  as-set object** - The creation of a as-set object is required only if we provide or plan to provide transit to another ASN. The ASN or as-set object of each of those ASN's should be listed as members of the object. Our ASN must also be a member of the object.
5. **Perform a query of our objects** - We should perform a query of our objects to confirm our they accurately reflect our network. Below are some common queries to perform. For more information refer to appendix B of the IRRd user guide at  [http://irrd.net/irrd-user.pdf](http://irrd.net/irrd-user.pdf).
	1.  Does a query of objects with our ASN return all of our IP space?
    -   whois -h <IRR server> \!gAS<our ASN>
    -   whois -h <IRR server> \!6AS<our ASN>
	2.  Does a query of our AS set expand to the ASN’s of all our downstreams?
    -   whois -h <IRR server> \!i<our AS-set object>,1
6. **Notify your peers** - We should notify all our peers and service providers which IRR database we maintain our objects and what object they should use to build filters.

### Turning up eBGP Peering

Enable Logging

#### Testing and Validation

##### Peer Establishment Issues

Is the Local AS configured correctly? Is the remote-as assigned correctly? Does your password match?  

##### Verify IP connectivity
* Check the routing table
* Use ping/trace to verify two way reachability
* Remember to allow TCP/179 through edge filters

##### Trying to load-balance over multiple links to the eBGP peer?
* Use extended pings to test loopback-to-loopback connectivity
* Validate Neighbor configuration for increased TTL

##### Be very careful with Multi-hop eBGP

* Check IP connectivity (local and remote routing tables)
* Remember to source updates from loopback
* Watch for filters anywhere in the path
* TTL must be at least 2 for eBGP-Multi-hop between directly connected neighbors
* Use TTL value carefully

##### Flapping Peer

* Validate Path MTU 
* Confirm the interfaces used for Peering are error free and the path is stable 
* Confirm the state of the remote router BGP process unstable, restarting 
* Ensure that Traffic Shaping & Rate Limiting parameters are not impacting the control plane

##### Peer Dropped after initial Establish

* Confirm MAX-Prefixes

## BCOP Conclusion / Summary

***NEEDED***
 [Any concluding remarks pertaining to the BCOP – no length limit]

**Bibliography:**
* "[RFC 4271](http://tools.ietf.org/html/rfc4271)  - A Border Gateway Protocol 4 (BGP-4)."  [RFC 4271](http://tools.ietf.org/html/rfc4271)  - A Border Gateway Protocol 4 (BGP-4). Accessed October 07, 2014. http://tools.ietf.org/html/rfc4271.
* "One Step Consulting - BGP Community Guides." One Step Consulting - BGP Community Guides. Accessed October 07, 2014. http://www.onesc.net/communities/.

