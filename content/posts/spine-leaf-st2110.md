---
date: 2026-04-02T00:00:00Z
draft: false
categories:
  - Broadcast Technology
  - AoIP
tags:
  - ST 2110
  - AES67
  - IP
  - Audio Engineering
  - Networking
title: "Spine-Leaf Network Topology in ST 2110 Broadcast Facilities"
description: "Spine-leaf is the network architecture that underpins most serious ST 2110 deployments. Understanding why it exists, how it behaves, and where it creates constraints is useful for anyone designing or troubleshooting AoIP systems."
cover:
  image: spineleaf2110.png
  alt: "Spine-leaf network topology diagram showing Red and Blue fabrics for ST 2022-7 redundancy"
---

When a broadcast facility moves to ST 2110, the network stops being background infrastructure and becomes a core part of the signal path. The choice of network topology has direct consequences for latency, redundancy, scalability, and how well PTP and multicast behave. Spine-leaf has become the dominant architecture for serious ST 2110 deployments, and understanding why - and what it asks of you in return - is worth the time.

## What Spine-Leaf Is

Spine-leaf is a two-tier network topology. Every leaf switch connects to every spine switch. No leaf connects directly to another leaf, and no spine connects directly to another spine. All traffic between devices on different leaf switches travels up to the spine layer and back down - always the same number of hops, regardless of which leaves are involved.

This is a fundamentally different architecture from the traditional three-tier model (access, distribution, core) that most broadcast engineers will have encountered in corporate or older facility networks. In a three-tier network, the number of hops between two devices varies depending on where they are in the hierarchy. In spine-leaf, it does not.

Devices - audio endpoints, video servers, control systems, anything on the network - connect to leaf switches. The spine switches carry traffic between leaves. You scale the network by adding leaves (more device connections) or adding spines (more bandwidth between leaves).

## Why ST 2110 Suits Spine-Leaf

ST 2110 puts significant demands on a network. High-bandwidth uncompressed video and audio streams, multicast traffic to potentially many receivers simultaneously, PTP synchronisation that requires stable and predictable delay, and the dual-path redundancy of ST 2022-7 all need a network that behaves consistently and predictably.

Spine-leaf delivers this in several ways.

**Consistent latency.** Because every inter-leaf path is the same number of hops, latency between any two points on the network is predictable. There are no long, winding paths through multiple distribution switches with variable queuing behaviour. For PTP, this is significant - the delay differential between the two paths needs to be small and stable for ST 2022-7 merging to work correctly, and for PTP offset calculations to remain accurate.

**Equal-cost multipathing (ECMP).** With multiple spine switches, there are multiple equal-cost paths between any two leaves. Traffic can be distributed across all of them simultaneously, which multiplies effective bandwidth and means a single spine switch failure does not take down inter-leaf connectivity - traffic redistributes across the remaining paths.

**Non-blocking design.** A properly specified spine-leaf fabric provides enough spine bandwidth that no leaf-to-leaf path is bandwidth-constrained. For ST 2110, where a single uncompressed HD video stream can consume 3 Gbps and a facility might have dozens of concurrent streams, this matters considerably.

**ST 2022-7 alignment.** The Red and Blue network model maps naturally onto spine-leaf. Two independent spine-leaf fabrics - separate physical switches, separate cabling - provide the genuinely independent paths that ST 2022-7 requires. Each fabric is its own spine-leaf network; devices connect to both simultaneously.

## Key Design Considerations

**Oversubscription ratio.** Leaf switches typically have more downlink bandwidth (to devices) than uplink bandwidth (to spines). The ratio of downlink to uplink capacity is the oversubscription ratio. A 1:1 ratio is non-blocking - no leaf port can ever be starved of uplink capacity. Higher ratios reduce cost but introduce the risk of congestion under peak load. For ST 2110 media networks handling uncompressed streams, low oversubscription ratios are important. For control and management networks, higher oversubscription is usually acceptable.

**Multicast handling.** ST 2110 relies heavily on IP multicast. Multicast traffic needs to be managed carefully - if it floods to all ports on all leaves, it will consume significant bandwidth on links that don't need it. **IGMP snooping** on leaf switches ensures multicast is only forwarded to ports where a receiver has expressed interest. **PIM (Protocol Independent Multicast)** on the spine layer handles multicast routing between leaves. Getting multicast right is one of the more complex parts of a spine-leaf ST 2110 design and worth validating thoroughly before going live.

**PTP boundary clocks.** In a spine-leaf fabric, PTP grandmaster messages need to reach all endpoints accurately. Running boundary clocks on the leaf switches - so each leaf re-generates PTP rather than passing it transparently - keeps the number of devices chasing the grandmaster manageable and improves synchronisation accuracy across the fabric. Transparent clocks on the spines handle the inter-leaf portion. This is the recommended model for larger deployments.

**QoS.** Differentiating media traffic from control and management traffic using DSCP marking and per-hop behaviour on the switches ensures that time-sensitive audio and video streams are not delayed by bulk data transfers or management traffic competing for the same queues.

## Advantages

**Predictable, low latency.** Fixed hop count and non-blocking bandwidth mean media streams behave consistently. There are no surprise congestion events caused by asymmetric paths or overloaded distribution switches.

**Horizontal scalability.** Adding capacity is straightforward - add leaf switches for more device connections, add spine switches for more inter-leaf bandwidth. You do not need to redesign the network to grow it.

**Fault tolerance.** Multiple spine switches mean no single spine failure takes the network down. Combined with ST 2022-7 dual fabric, the architecture supports the resilience requirements of live broadcast.

**Operational simplicity.** Uniform hop count and consistent design make troubleshooting more straightforward than a complex three-tier hierarchy with variable paths. When something goes wrong, the failure domain is easier to isolate.

## Disadvantages

**Cost.** Spine-leaf requires high-performance switches at every tier. The non-blocking bandwidth requirements, the need for hardware PTP support throughout, and the duplication for ST 2022-7 redundancy mean the capital cost of a spine-leaf ST 2110 fabric is significant. This is not an architecture built from commodity switches running default configuration.

**Complexity.** Configuring ECMP, multicast routing, PTP boundary clocks, IGMP snooping, QoS policies, and dual-fabric redundancy across a spine-leaf fabric requires network engineering expertise that broadcast engineers have not traditionally needed to have. Facilities making the ST 2110 transition often underestimate this.

**Scale limitations on the spine.** The number of leaf switches is constrained by the port count on the spine switches. In very large facilities, this can become a limiting factor. Larger deployments sometimes use a super-spine layer, which adds a third tier and reintroduces some of the variable-path characteristics that spine-leaf was designed to avoid.

**East-west traffic assumption.** Spine-leaf is optimised for traffic between devices at the same tier - which is exactly the pattern for ST 2110 media flows. It is less well suited to heavy north-south traffic (in and out of the facility to external networks), which tends to require additional architecture at the boundary.

## From an AoIP Perspective

For audio specifically, the spine-leaf fabric is often more network than the audio streams strictly require. A 64-channel AES67 stream at 1ms packet time consumes around 80 Mbps - significant, but well within the capacity of a modern 10GbE or 25GbE leaf port. The network demands of audio are modest compared to uncompressed video, which is what really drives the specification of the spine layer.

What audio does care about is latency stability and PTP accuracy, both of which spine-leaf delivers well. Variable latency caused by congestion or asymmetric paths shows up as jitter in AES67 streams, which increases the required playout buffer and can cause synchronisation issues at high channel counts. The predictable behaviour of a well-designed spine-leaf fabric removes these variables.

The practical implication for audio engineers working in ST 2110 facilities is that the network is almost certainly designed to a specification driven by video requirements, and audio fits comfortably within it - provided PTP is configured correctly and multicast is managed properly. Those two things are where audio problems in spine-leaf ST 2110 networks most commonly originate, and they are worth understanding in detail before commissioning any significant AoIP system on this infrastructure.
