Best Current Operational Practice: 
DDoS / DoS Response
=============

```
Date: ?
Subject: What to do in response to DDoS / DoS attack
Shepard: Yardiel Fuentes
Subject Matter Expert(s) (SME): Rich Compton, Prabhu Gurumurthy, John W, Damon Fortune, Yardiel Fuentes
Status: Draft

This document is intended to be original content authored by the Global Network Engineering Community 
(GNEC) "at-large" in an organized, democratic, "bottoms-up" approach.
```

## BCOP Summary  / Appeal

This BCOP aims to be a guide of practices that have proven effective in production network environments of what to do before, during and after a DDoS/DoS attack. These practices are based on experiences and tactics that have worked satisfactorily.

## BCOP Background / History

NANOG members have faced, and continue to deal with Denial of Service attacks in their network infrastructure. Denial of Service (DoS) or Distributed Denial of Service (DDoS) attacks’ main purpose is to take a significant toll on a network’s role and capacity to serve and satisfy its intended purpose. As these DDoS/DoS attacks continue to grow in quantity while becoming more sophisticated, network operators find themselves being willing or unwillingly, Attack defenders. An informal measure of this involvement is the number of DDoS/DoS attack-related discussions and perspectives shared in NANOG forums. This BCOP is the result of as a grass-roots effort to gather and disseminate relevant, collective best practices in order to assist network operators in becoming more effective DoS attack defenders. While specific details of DDoS/DoS attacks often do vary, these DDoS/DoS Attack Mitigation Best Common Operational Practices present common specific recommendations and concise information to network operators for DDoS/DoS mitigation.

## BCOP Specifics

### What To Do Prior To a DDoS/DoS Attack

Some widely acceptable functionality that should be implemented in networks in order to mitigate DDoS/DoS attacks are listed below:

1.  Configure ASIC-based packet filtering in all your network interfaces of your routers and switches. These are often referred to as Access-Control-Lists (ACLs) or Firewall Filters. These are essentially applicable at the network gear line card interfaces (often called “forwarding plane”).
    1.  Be aware that it may be appropriate in your network to only accept certain protocols into your network while others need to be denied (especially UDP based protocols, which are quite prevalent in amplification attacks)
        1.  Some common traffic used for DDoS attacks that ISPs block inbound are UDP source port 19 (Chargen) and UDP source port 17 (QOTD).
        2.  ISPs frequently block NTP monlist reply packets with the maximum 6 IP addresses populated in the payload. These packets are frequently used in DDoS amplification attacks. To block these, match UDP traffic with a source port of 123 and a size of 468 bytes.
        3.  Even though there are DNS reply packets which can be larger than 512 Bytes such as those implementing EDNS, it is worth accounting for valid DNS replies larger than 512 Bytes in your network – these packets are commonly found in DNS DoS Amplification attacks. Many vendor firewalls by default limit DNS packets to 512 Bytes even though this is configurable. You could use the “dig” *nix command to explore such details as shown below:  [File:Dig-cmd.jpg](http://nabcop.org/index.php?title=Special:Upload&wpDestFile=Dig-cmd.jpg "File:Dig-cmd.jpg")
    2.  Be aware that it may be appropriate in your network to rate limit the amount of traffic for a given protocol using a specific interface or port.
        1.  Some common traffic used for DDoS attacks that ISPs rate limit are UDP source port 161 (SNMP), UDP source port 1900 (SSDP), and UDP fragments usually seen with Chargen and DNS reflection attacks.
    3.  Be aware that it may be appropriate in your network to accept traffic only from some acceptable prefix subnets as source addresses at each router or switch,
    4.  Make sure to disallow all packets with your IP/subnet as the source.

2.  Make sure to have proper graphing and trending enabled on all service provider interfaces and all downstream critical devices. This recommendation is a highly effective in rapid identification of unusual network traffic patterns.

3.  Implementation of Reverse Path Forwarding checking, particularly, in your upstream/external interfaces (i.e. rpf-check functionality) help alleviate the potential for receiving traffic from unidentified, spoofed IP sources.

4.  External monitoring of the services to monitor and notify of the service impact and internal monitoring based on the unusually large increase in interface counters and/or errors.

5.  Configure packet filtering to protect your network control and forwarding planes in each of your nodes. If your network gear was designed to show distinct separation of the control and forwarding planes (check Figure below), be aware that it is appropriate to implement packet filtering using ACLs or Firewall filters for all traffic before hitting the control and forwarding planes.  * _missing file_*

6.  Implement ingress filtering recommendations as presented in BCP38 and BCP84 for avoiding DDoS/DoS attacks using forged IP addresses behind an ISP service. Check the following URLs for details on ingress filtering specific methodologies in dealing with DDoS/DoS attacks:  [http://tools.ietf.org/html/bcp38](http://tools.ietf.org/html/bcp38)  and  [http://tools.ietf.org/html/bcp84](http://tools.ietf.org/html/bcp84)

7.  Consider disabling open recursion for your own name servers and exclusively accept DNS recursion from trusted sources. If external DNS servers are being used, testing these to check for open recursion would allow you to know if your network is exposed “by association”.

8.  Identify/mitigate customers with open relays. Use threat intelligence (for example from Open Resolver Project/Shadowserver) and/or scanning to determine which customers in your network have open relays that can be used for DDoS amplification. Mitigate these open relays by notifying customers of the issue and helping them resolve the issue, applying ACLs, and/or placing customers in a walled garden.

9.  It is also an option to consider implementing DDoS/DoS mitigation subscription services offered by ISPs or 3rd party companies specialized in dealing with DDoS/DoS mitigation. There are different strategies offered by these subscription services. The capability to spread and deliver the victim’s traffic across multiple sites efficiently is a common implementation example of these services.

#### Implement a DDoS/DoS Detection and Mitigation Solution

1.  Solution should passively collect information about network traffic using exported network flow information from routers or network tap data. There are various options for collecting this information by implementing network telemetry features (such as sflow, netflow, cflowd, jflow, etc) or port mirroring, allowing for later “tcpdump like” data analysis. It should be noted that there are differences among netflow, sflow, jflow, etc which are worth considering depending on your needs and preferences.

2.  Establish baselines of normal traffic to identify when DDoS/DoS flood traffic is occurring. This can be rather challenging given the need for close familiarity with traffic patterns but it can prove highly effective in detecting abnormalities in your network.

3.  Configure the DDoS/DoS mitigation solution to have the ability to selectively divert traffic destined to a victim towards one or more DDoS mitigation devices. These DDoS/DoS mitigation devices should have the ability to be configured to block traffic that is identified as attack traffic and permit valid traffic.

4.  The DDoS/DoS mitigation devices should have the ability to “on-ramp” the valid traffic back down to the customer network with minimum incurred traffic delay/loss.

5.  An option for the DDoS mitigation device is to have the DDoS detection/mitigation solution use BGP Flowspec to push out a NLRI (Network Layer Reachability Information) block (or rate limit) policy to the edge routers to mitigate the DDoS attack traffic.

6.  Since a large percentage of DDoS/DoS attacks lasts less than 30 minutes, it is preferable that the DDoS/DoS Detection/Mitigation solution be capable of automatically diverting victim traffic in mitigating the attack traffic. This approach allows for less human involvement needed to mitigate the attacks while speeding up the attack mitigation.

7.  In general, DDoS/DoS attacks can take different forms. Some of these attacks leverage spoofed information (such as TCP win 0 and synfloods), others are single-source UDP such as LOIC-based attacks and involved large scale botnets; evaluation of attack limitations and guarantee levels offered by a DDoS/DoS solution is an important differentiator.

### What To Do During a DDoS/DoS Attack

1.  Identification of the attack. This identification involves observation of interface counters/network flow data/network tap data. Therefore, on-going interface traffic monitoring is important; preferably in 10 - 30 second intervals. This interval can arguably be made even smaller if automatic mitigation solutions are in place. This type of interface traffic monitoring should be carried out with available external monitoring systems if at all possible in order to reduce a node’s system load.

2.  Tune the existing edge ACL to block the attack traffic. If available, use BGP Flowspec rules on the network devices to thwart attacks at the egress devices.

3.  Divert (Off-ramp) traffic to the DDoS/DoS victim to DDoS/DoS mitigation device(s), usually using BGP. Configure the DDoS mitigation devices to block attack traffic and permit valid traffic. On-ramp the valid traffic back down to the customer.

4.  Contact the upstream ISP(s) and provide them the details of the attack traffic. Request that they block the attack traffic before it reaches your network.

5.  Monitor the backend services to confirm that they are running an available when the DDoS/DoS subsides.

### What To Do After a DDoS/DoS Attack

After a DDoS/DoS attack has been mitigated or it has ceased, the following are some recommended actions to take (in no particular order):

1.  Create a post-mortem traffic variation analysis to determine the breath and scope details of the protocol exploited by the DDoS/DoS in your network. The goal is to clearly identify the details of how the DDoS/DoS attack was effective in impacting your network including the attack intensity, duration, leveraged vulnerability, and sources. This information provides the attacks anatomy for dealing with potential future repeats.

2.  A controversial step is the decision of reporting a DDoS/DoS attack to the authorities. If the decision is to report the attack, the IC3 ([http://www.ic3.gov/default.aspx](http://www.ic3.gov/default.aspx)) and the FBI are two agencies to consider. However, the FBI as well as other governmental enforcement authorities have been deemed slow-moving and will likely not be able to help as much as expected (plus the FBI may redirect you to report it to the IC3).

3.  A community-wide beneficial step is to share details as the DDoS/DoS attack victim with the NANOG community in order to grow the community’s collective knowledge and awareness. This can be done with a simple email to the NANOG list given that these messages are archived and can be easily searched.

## General Recommendations or Pointers

This section seeks to share collected community-wide information relevant at all previous sections:

1.  DDoS/DoS attacks have a wide range of type-value characteristics; consequently, knowledge of your network valid traffic patterns can be very useful. Some value references are listed simply for general reference:
    1.  Attack intensity typically can have a wide range from 1G up to 400G with higher values not unrealistic. NOTE: attack intensity measurements: packets-per-second or bits-per-second, can be relevant metrics in different DDoS/DoS scenarios.
    2.  Number of DDoS/DoS attack sources can have a wide range in quantity from a handful to millions of source attackers. These sources can be willing or unwilling source attackers globally and can also include game service providers IP ranges.
    3.  Attack durations often oscillate from seconds to minutes. Some sync and sync/ack based attacks can be extremely intense and quickly rendered a 10G multicore hardware server NICs unavailable and drawn in interrupt calls within seconds.

2.  If DDoS/DoS attacks captured telemetry information is available (e.g. sflow, netflow, jflow, etc) as mentioned previously during a UDP-based reflection/amplification DDoS/DoS attack, notably DNS, SNMP, charged, SSDP, tftp, or other UDP-based protocols utilizing network-based fragmentation (NTP does not), it is likely that statistics reporting traffic sourced from and destined for UDP/0 may be reported on the derived traffic statistics. It should be noted that non-initial UDP fragments are generally reported as UDP/0 by multiple flow telemetry exporters and collection/analysis systems; therefore, network operators should be aware of this caveat and take it into account in analyzing tools and techniques to mitigate UDP-based attacks.

3.  There are DDoS/DoS attackers that often email their victims asking for ransom or compensation to stop the attack. It can be rather controversial whether or not to comply with such compensation request. In general, the consensus is to never pay any ransom or compensation to attackers, regardless of how small or large the requested amount is.

4.  During the process of identifying the attack’s source ASN and IP addresses, keep in mind it is often the case that these may have been sub-allocated. A follow-up step after identifying the source ASN and IP addresses is to contact the abuse contact/email listed in the whois allocated space with the attack details and logs (include as much detail as possible in your original contact email to increase the chances of a prompt response). However, be warned that the effectiveness of this step is often arguable since it is dependent on the space owners’ abuse-email management.

5.  When dealing with DNS reflection/amplification attacks in which large, unsolicited DNS responses are typical, previously mentioned techniques such as Source-based/remotely-triggered Blackholing (S/RTBH) or similar filtering techniques working in conjunction with upstream peers can be effective as well as FlowSpec internal network implementation. S/RTBH basic component is uRPF functionality mentioned previously in this document.

6.  It should be noted that mitigating outbound DDoS or DoS attacks should also be part of the overall mitigation strategy. Outbound attacks essentially make “bots” of vulnerable intermediate infrastructure. Constant monitoring of processes and services using outbound connections and familiarity with such processes should be part of any outbound attack strategy.

7.  For tool recommendations or comments, please review search results in NANOG email archives under DDoS or DoS subjects – This BCOP does not provide any recommendations or endorsements given its vendor-agnostic approach. However, there are open source and/or community-driven DDoS tools and projects listed as examples below but whose merits and effectiveness are left unevaluated in this BCOP, such as:
    1.  FastNetMon. It is an open source and community driven tool which can be found at GitHub:  [https://github.com/FastVPSEestiOu/fastnetmon](https://github.com/FastVPSEestiOu/fastnetmon)
    2.  Mod Security Apache Module. It is an open source web application firewall which can be found at:  [https://www.modsecurity.org](https://www.modsecurity.org/)
    3.  Advanced Policy Firewall (APF). It is a linux project and more details can be found at:  [https://www.rfxn.com/projects/advanced-policy-firewall/](https://www.rfxn.com/projects/advanced-policy-firewall/)
    4.  (D)DoS Deflate. It is a lightweight bash shell script and more details can be found at:  [http://deflate.medialayer.com](http://deflate.medialayer.com/)

If you find other open-source, non-commercial tools that the NANOG community should be aware of, please notify NANOG BCOP committee at: bcop-support@nanog.org or this BCOP shepard (yardiel@gmail.com).

## BCOP Conclusion / Summary
Needed.

