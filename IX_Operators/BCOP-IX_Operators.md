Best Current Operational Practice: 
IX Connections
=============
```
Date: 2023-08-10
Shepard: NEEDED
Subject Matter Expert(s) (SME): NEEDED
Status: Draft
BCOP Subject: Public Peering IX Operations

This document is intended to be original content authored by the Global Network Engineering Community 
(GNEC) "at-large" in an organized, democratic, "bottoms-up" approach.
```
  
## BCOP Summary / Appeal
This document describes policy and network components that comprise the best current operational practices for running a public Internet Exchange (IX) point. It also provides general configuration parameters and guidelines, but does not include vendor-specific configuration information.

## BCOP Background / History
There are a multitude of exchange points around the world each of which has a set of independent providers using a common exchange fabric to connect and pass traffic to one another. Although there are always specific configuration requirements for particular exchange points, there are also common configuration parameters that, if used, will add to the stability of any exchange.

This BCOP lays out the guidelines that should apply in most situations, providing a foundation for Exchange-specific requirements, but not superseding them. IX operators may opt to diverge from this BCOP based on their specific constraints and considerations.

This BCOP is specifcally about operation an IX, and meant to be used in addition to the [IX Connections](https://github.com/Open-IX/BCOP/blob/main/IX_Connections/BCOP-IX_Connections.md) BCOP.

## BCOP Specifics

This BCOP is divided into four sections:
- [IX Connections](#ix-connections)
	- [BCOP Summary / Appeal](#bcop-summary--appeal)
	- [BCOP Background / History](#bcop-background--history)
	- [BCOP Specifics](#bcop-specifics)
		- [Section 1: Use of ASNs](#section-1-use-of-asns)
	- [BCOP Conclusion / Summary](#bcop-conclusion--summary)

This BCOP uses the following standard language throughout.
1) Exchange – a common term for an Internet Exchange, Public Peering Point, Exchange fabric, etc.
2) Participant – a common term for any entity or entities connecting into the Exchange.

### Section 1: Use of ASNs

1) **Route server ASNs** – A globally unique ASN SHOULD be used for the route servers of each IX. This ASN SHOULD NOT be used for anything besides route server operation.

## BCOP Conclusion / Summary

TBD
