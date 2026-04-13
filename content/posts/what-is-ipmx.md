---
date: 2026-04-11T12:00:00Z
draft: false
categories:
  - Broadcast Technology
  - AoIP
tags:
  - IPMX
  - ST 2110
  - NMOS
  - AoIP
  - IP
  - Networking
  - Pro AV
title: "What Is IPMX, and Why Could It Change AV over IP?"
description: "IPMX is an open standard for professional AV over IP, built on the same foundations as ST 2110 and NMOS. It adds what the Pro AV market actually needs - HDCP support, compressed transport over 1GbE, and a unified control layer - and it could finally end the proprietary fragmentation that has defined AV over IP for years."
cover:
  image: ipmx_logo.png
  alt: "IPMX - IP Media Experience"
---

The broadcast industry spent the better part of a decade converging on ST 2110 and NMOS as the open standards for professional media over IP. The Pro AV industry - corporate, higher education, live events, hospitality, digital signage - has been having a parallel conversation, mostly in private, and mostly arriving at proprietary conclusions.

SDVoE, Dante AV, NDI, AVoIP chipset ecosystems from various manufacturers: each solves the AV distribution problem, none of them solve it in the same way, and none of them interoperate with each other or with broadcast infrastructure. If you have ever tried to connect a corporate AV system to a broadcast facility, or route signal between a proprietary HDMI-over-IP encoder and an ST 2110 infrastructure, you will understand the problem immediately.

IPMX is the answer to that problem. Whether it becomes the answer the industry actually adopts is a different question.

## What IPMX Is

IPMX stands for Internet Protocol Media Experience. It is a framework of open standards developed under the Alliance for IP Media Solutions (AIMS), building on the broadcast standards that already exist rather than inventing new ones.

The foundation is SMPTE ST 2110, the same transport standard used in broadcast facilities for uncompressed professional media over IP. On top of that sits NMOS - specifically IS-04 for device discovery and registration, and IS-05 for connection management. IEEE 1588 PTP provides the timing reference. AES67 handles audio.

If that sounds like a broadcast media network, it largely is - intentionally so. The premise of IPMX is that the broadcast industry has already solved the hard engineering problems of transporting professional media over IP with full timing synchronisation and multi-vendor interoperability. Rather than the Pro AV industry solving those problems again with proprietary technology, IPMX extends the broadcast solution to cover the specific requirements of AV environments.

The extensions are what make it relevant to Pro AV rather than just another name for what broadcast facilities already do.

## What IPMX Adds

### JPEG XS Compression

ST 2110 is an uncompressed transport standard. An uncompressed 1080i50 video stream consumes around 1.5 Gbps - which requires 10GbE infrastructure as a minimum and typically 25GbE or higher in production environments. Broadcast facilities are built for this. Corporate AV environments are almost universally 1GbE, because that is what standard IT infrastructure provides.

JPEG XS changes the equation. It is a visually lossless compression codec designed specifically for low-latency professional applications. The encode/decode process is fast, which means it behaves like an uncompressed signal for all practical purposes in AV applications. A 1080i50 stream compressed with JPEG XS can be transported comfortably on a 1GbE link.

IPMX incorporates JPEG XS as the primary compressed transport option, defined in SMPTE ST 2110-22. This is what makes IPMX viable on standard IT infrastructure rather than requiring a purpose-built high-bandwidth network. For Pro AV, it is arguably the most important technical decision in the specification.

### HDCP Support

HDCP (High-bandwidth Digital Content Protection) is the content protection mechanism used with HDMI and DisplayPort. In broadcast, HDCP is largely irrelevant - broadcast signals are not HDCP-protected. In Pro AV, it is unavoidable. Corporate AV systems routinely handle laptop presentations, streaming content, and video conferencing outputs that carry HDCP. Any AV distribution system that cannot handle HDCP will fail in the field.

Proprietary AV over IP systems have handled HDCP in various ways, mostly by tunnelling it through their own encrypted transport and decrypting at the output. IPMX defines an open mechanism for HDCP key exchange and content protection across an IP network, which is one of the technically complex additions the AIMS Pro AV Working Group has had to solve. Getting this right is important - any gap in HDCP compliance will prevent IPMX systems from being used with protected content sources.

### Unified Control with NMOS

Perhaps the most significant long-term implication of IPMX is the control layer. Because IPMX uses IS-04 and IS-05, an IPMX device is discoverable and controllable by the same NMOS control tools used in broadcast facilities. A control application that can manage an ST 2110 routing matrix can, in principle, also manage an IPMX AV distribution system.

This has real consequences for facilities that span both worlds. A broadcast facility with a corporate AV wing, a sports venue with both production and presentation infrastructure, a conference centre that hosts live events - all of these currently require separate control systems for broadcast and AV components. IPMX and NMOS together make a unified control layer possible.

## The Problem IPMX Is Solving

The Pro AV industry's AV over IP market has followed a familiar pattern. A genuine technical problem - distributing video and audio around a building or campus over IP rather than running dedicated cables - attracted multiple vendors who each solved it their own way. The result is a market where SDVoE (backed by Semtech), Dante AV (Audinate), NDI (Vizrt), and several others each have working solutions that do not interoperate with each other.

For an integrator or end customer, this means making a platform choice that locks in a vendor ecosystem. Switching later requires replacing the infrastructure. Adding a device from outside the ecosystem is not straightforward. The economics favour the platform vendor, not the customer.

The AIMS whitepaper from 2019 makes the parallel with broadcast's earlier history explicit. The broadcast industry went through the same proprietary fragmentation before converging on open standards. AIMS' position - supported by over 100 member companies - is that the Pro AV industry should skip the proprietary phase and go directly to the open standard.

The argument is straightforward: if every IPMX device uses IS-04 for registration, IS-05 for connection management, ST 2110 for transport, and JPEG XS for compression, then any IPMX encoder works with any IPMX decoder, and any NMOS control application can manage the whole system. No platform lock-in. No royalties. No proprietary chipset dependency.

## Why It Could Be a Game-Changer

**Convergence of broadcast and Pro AV.** For facilities that operate in both spaces, a common open standard eliminates a class of integration problems that currently require custom bridging solutions or proprietary translation hardware. A broadcast production network and a corporate AV distribution system built on IPMX share the same control plane and can be managed together.

**End of proprietary fragmentation.** The proprietary AV over IP platforms are entrenched, but they share a structural weakness: any individual platform's ecosystem is limited to that platform's participants. An open standard has no such ceiling. If IPMX achieves broad adoption, the total number of interoperable products will eventually exceed what any proprietary platform can offer.

**Standard IT infrastructure.** Because JPEG XS enables IPMX on 1GbE networks, deployment does not require specialist high-speed switching. The network is standard IT infrastructure. This reduces capital cost, simplifies network management, and allows AV systems to coexist on existing networks rather than requiring dedicated media infrastructure.

**Multicast scaling.** Like ST 2110, IPMX uses IP multicast. A single IPMX stream can be received simultaneously by multiple endpoints without the source generating multiple copies. At scale - a large venue, a campus distribution system, a multi-room AV installation - this has significant bandwidth and infrastructure advantages over unicast-based approaches.

**Professional quality and timing.** IPMX inherits ST 2110's approach to timing and synchronisation. PTP-synchronised streams mean audio and video remain in sync across the network, and multiple displays or speakers in different locations can be driven synchronously. For live events and corporate AV in particular, this is a meaningful quality improvement over systems that rely on software buffering for synchronisation.

## The Honest Challenges

IPMX is not yet a mature, widely-deployed standard. Several practical challenges remain.

**Adoption takes time.** AIMS demonstrated a working multi-vendor IPMX interoperability proof of concept at ISE 2019. The standard has developed significantly since then, but the ecosystem of shipping, tested, certified IPMX products is still growing. Proprietary platforms have years of deployment history, established support networks, and large installed bases.

**HDCP complexity.** Implementing HDCP correctly across an IP network is technically demanding. Getting it to work reliably in all configurations - different HDCP versions, different source devices, different network topologies - requires careful engineering. Interoperability testing across vendors for HDCP-protected content will be an important proving ground for the standard.

**Network configuration.** IPMX inherits ST 2110's requirements for multicast configuration, IGMP snooping, and PTP on the network. While standard 1GbE switches suffice for bandwidth, they still need to be correctly configured for multicast and timing. In a corporate AV context, this is more configuration than a typical IT team is used to managing.

**The control plane still needs implementation.** NMOS is the right choice for the control layer, but IS-04 and IS-05 implementation across a fragmented vendor ecosystem takes time. The interoperability that NMOS promises requires consistent, validated implementations - and that means conformance testing and certification, which is a process, not just a specification.

## Where It Sits in the Landscape

For broadcast engineers, IPMX is worth understanding as the Pro AV industry's path toward the same open standards infrastructure that broadcast has been building for the last decade. The more interesting near-term implication is convergence - the possibility of a broadcast production environment and an AV distribution system sharing infrastructure and a control layer without requiring custom integration.

For Pro AV engineers and system integrators, IPMX represents a way out of the platform lock-in cycle. The transition will not happen overnight, and existing proprietary systems will not disappear quickly. But the direction is consistent with how the broadcast industry's own IP transition developed: initial fragmentation, followed by standards convergence driven by customers who valued interoperability more than any single vendor's feature set.

The building blocks are solid. SMPTE ST 2110, NMOS, AES67, PTP, JPEG XS - these are proven, well-supported technologies. IPMX is not asking the industry to adopt something new; it is asking the industry to extend something that already works in broadcast into the adjacent AV market. That is a more achievable proposition than starting from scratch, and it is why IPMX deserves to be taken seriously even at this stage of its development.
