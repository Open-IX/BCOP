Best Current Operational Practice: 
IPv6 Peering
=============

```
Date: 12 October 2012
Subject: How to configure BGP peering sessions for IPv6
Shepard: Chris Grundemann
Subject Matter Expert(s) (SME): Zaid Ali, Bill Blackford, Chris Grundemann, Aaron Hughes, Darius Jahandarie, Jonathan Lassoff, Joe Provo, Ren Provo, Brandon Ross, Michael K. Smith
Status: Draft

The content here within is intended to be original content authored by the Global Network Engineering
Community (GNEC) “at-large” in an organized, democratic, “bottoms-up” approach.
```

## BCOP Summary / Appeal

This BCOP aims to provide general IPv6 Peering and Transit guidelines that can be followed by any network operator when planning and implementing any IPv6 Peering/Transit relationship. The primary focus is on understanding BGP peering and filtering.

## BCOP Background / History

We (the GNEC) have made many mistakes with IPv4 Peering and Transit configurations and operational practices. As operators begin turning up more and more IPv6 E-BGP sessions with peers and transit providers, there is an opportunity to do things right from the beginning. While the details of these peering and transit relationships can be varied and specific, the technical realities remain largely the same. These technical realities inform the best practices listed here.

## BCOP Specifics

### IPv6 Only
Establish new, IPv6-Only peering sessions parallel to existing IPv4 peering. Individual IPv4 and IPv6 BGP peering sessions should be established between all BGP neighbors, particularly eBGP peers. While it is possible to use Multiprotocol BGP (MP-BGP) [Bates, T., Chandra, R., Katz, D., and Y. Rekhter, “Multiprotocol Extensions for BGP-4”,  [RFC 4760](http://tools.ietf.org/html/rfc4760), January 2007] to carry IPv6 Network Layer Reachability Information (NLRI) over existing (or new) IPv4 BGP peering sessions, this is not recommended. Both BGP sessions MAY use the same logical circuit, or, a new port MAY be used for IPv6 (separate physical or logical connections is NOT a requirement).

_Note: The IPv6 topology should be parallel to but independent of IPv4._  ([http://blog.caida.org/best_available_data/2012/06/04/ipv6-what-could-be-but-isnt-yet/](http://blog.caida.org/best_available_data/2012/06/04/ipv6-what-could-be-but-isnt-yet/))

[File:P-t img 1.jpg](http://nabcop.org/index.php?title=Special:Upload&wpDestFile=P-t_img_1.jpg "File:P-t img 1.jpg")

This maintains independent IPv6 and IPv4 topologies, rather than tying the two together unnecessarily. It prevents black holing of IPv6 traffic in the event of a protocol outage because the IPv6 session goes down when IPv6 reachability is lost. When an IPv4 BGP session carries IPv6 NLRI, IPv6 routes are only withdrawn if IPv4 connectivity is lost. Independent BGP sessions also facilitate protocol specific maintenance because the IPv4 and IPv6 sessions don’t affect each other (e.g. IPv6 can be “bounced” without effecting IPv4 and vice verse). Finally, establishing new, IPv6-only peering creates better operational clarity. It allows IPv4 and IPv6 configuration stanzas to be independent and easily recognizable.

### Filter Routes
 ([http://www.space.net/~gert/RIPE/ipv6-filters.html](http://www.space.net/~gert/RIPE/ipv6-filters.html))

Route filtering methodology for IPv6 generally mirrors IPv4. You still have explicit filters, bogon & martian filters ([https://www.team-cymru.org/Services/Bogons/](https://www.team-cymru.org/Services/Bogons/)), maximum-prefix limits, prefix size filters, as-path filters, etc. As you might expect, the devil’s in the details. BGP route filtering can be separated into three high-level “themes:” Filtering customers, filtering Peers and Transit providers, and filtering your own routes.

#### Filter routes coming from your customers

The best practice for filtering customer routes is to use an explicit filter. Your filter should allow known customer network(s) and autonomous system numbers (ASNs), denying everything else. Building such explicit filters requires that your customer provide a list of their networks to you and to update that list as needed. This works especially well in IPv6 because the vast majority of customers won’t have more than one network and updates (new networks) should be infrequent if needed at all. Still, the authors recommend that you provide your customers a tool or procedure for making such updates when they are necessary.

_Note: One option is to require your customers to register their routes in an Internet Routing Registry (IRR)._  ([http://www.irr.net/](http://www.irr.net/))

The one remaining question is in regard to prefix size limitations. In IPv4 it’s common to allow prefix’ down to /32 (for black-holing, etc.). There are orders of magnitude more addresses in IPv6 however. A single /64 announced as individual /128s would more than fill all available routing table slots in your edge router. If the practice of accepting down to /128 is allowed in IPv6; you should only allow routes with a specific pre-arranged community or set of communities (e.g. <asn>:666) and you must ensure proper maximum-prefix limits! In any case, you must be sure to filter out all more specifics before announcing customer networks to your Peers and Transit providers.

_Note: You could filter at the /48 boundary if the customer needs to leverage more specifics (for multihoming, etc.). This should be done as an exception however, not as the default practice._

#### Filter routes coming from Peers & upstreams

There are two possible methods to filter routes from non-customer connected networks; use an IRR to generate explicit prefix filters, or configure a bogon filter and a maximum-prefix limit.

The preferred method is to generate prefix filters from an Internet Routing Registry (IRR). This requires that your peer or upstream register their announced networks in an IRR. Assuming that they are (they should be, see BCOP #3 below) you can use IRR data for tool-based filter generation ([http://irrtoolset.isc.org/](http://irrtoolset.isc.org/)). This method has one limitation: It’s as good as the data in the IRR. Luckily we have a chance to do this right from the beginning with IPv6.

If the network you need to filter does not use an IRR (and you can’t convince them to start), you are left using a bogon filter coupled with a maximum-prefix limit. The bogon filter rejects the unwise entries (reserved, small prefix, etc.) and the maximum-prefix limit protects against route overload. While not ideal, this is absolutely better than nothing – and sometimes you don’t have a choice.

#### Filter your own routes

There are three best practices that you should apply when filtering the routes you advertise out to connected networks.

First, only allow you and your customer’s address space and ASNs to be advertised. Ensure, through explicit filters, that you never announce any prefix or ASN for which you are not responsible (aka someone else’s network).

Second, do not deaggregate unnecessarily. In most cases you should not need to advertise anything than the single aggregate for each advertised network. When you do have reason to announce smaller prefixes, keep it to a minimum and always ensure that they are /48 and shorter (larger) only (longer than /48 will almost certainly be discarded anyway).

Finally, use communities to identify and filter routes within your network. The use of communities can greatly simplify the filtering process by allowing edge routers to understand more context about each prefix received. Some examples of what context communities can be used to convey are: Level of hierarchy (allows you to filter more specifics from deeper in the hierarchy without missing smaller top level prefix’), type of customer (BGP customers are more likely to multihome, so different filtering rules may be in order), use of network (there’s likely no need to advertise specific infrastructure prefixes), and locational information (allows regionalization for traffic engineering, etc.).

### Use an IRR

Internet Routing Registries (IRRs) are used in two basic ways, to publish your network’s information and to obtain similar information from other networks. As discussed in the last section, this exchange of route information facilitates the generation of tool-based / automated route filters for all participants. This precise and explicit filtering helps reduce route hijacking, both accidental and malicious. Further, the publication of this information also allows faster troubleshooting by giving you information on the origins of a route you may not otherwise have access to. In order for this to be of maximum value to all participants, the IRR information must be complete and accurate.

### Use IPAM

IPAM tools have historically consisted of spreadsheets and/or various commercial/home grown solutions. Since their express purpose is to track and monitor IP addresses, there has been plenty of industrial history for IPv4 - but there are some big distinctions when dealing with IPv6. As noted above, the sheer number of addresses is now greatly increased; so tracking allocations becomes even more important. When evaluating solutions to help manage IPv6, be sure to look for solutions that take on the subnetting component since having to do subnet calculations outside of a given IPAM tool defeats the purpose. This also ensures that you not only track assignments from your given allocation, but also track and manage the unassigned space as well. To have an IPv6 tool that doesn't take this into account invites errors with manual steps due to the more complex nature of IPv6 addresses.

When evaluating IPAM solutions, you will most likely also investigate how IPv6 affects your existing DNS/DHCP infrastructure. Most vendors of IPAM solutions also have products/integrations that address the DNS and DHCP realms (the industry term for this is DDI). The introduction of IPv6 and other technologies such as DNSSEC, have ensured that vendor solutions mature to accommodate the variety of infrastructures in production today - but IPv6 support should still be tested since there are still plenty of opportunities for data entry errors if the solution relies on manual edits/updates.

One key area to investigate is the level of integration possible with your current workflow. Is there a simple API that is easy to tie into your environment? Or does it require you to deploy new appliances to see the benefits of your IPAM tool? Ideally, you can identify solutions that combine some/all of these key features including Peering, Asset Management, DNS, and DHCP for even more time savings.

### Register in the PeeringDB

PeeringDB are, and how to contact them. Just like IRRs, the PeeringDB is used bi-directionally and grows in value the more it is used. Registering in the PeeringDB allows other networks interested in peering to find and contact you. Likewise, having others participate allows you access to the best peering information. ([https://www.peeringdb.com/](https://www.peeringdb.com/))

## BCOP Conclusion / Summary

We can do a better job with IPv6 than we did with IPv4. Let's get this right from the beginning!

 1. Establish new, IPv6-only peering sessions
 2. Use route filters
	 1. Filter routes coming from your customers
	 2. Filter routes coming from Peers & upstreams
	 3. Filter your own routes
3. Use an IRR
4. Use IPAM
5. Use PeeringDB

