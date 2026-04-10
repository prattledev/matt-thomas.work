---
date: 2026-04-08T12:00:00Z
draft: false
categories:
  - Broadcast Technology
  - AoIP
tags:
  - Networking
  - ST 2110
  - AES67
  - IP
  - Audio Engineering
  - Multicast
title: "IGMP in Broadcast Media Networks"
description: "IP multicast is the foundation of ST 2110 and AES67 transport. IGMP is the protocol that keeps it under control - without it, every multicast stream floods every port on every switch. Here is what it does, how it works, and what it means for a spine-leaf media network."
---

ST 2110 and AES67 both depend on IP multicast. A single uncompressed HD video stream can consume 3 Gbps. A facility running dozens of simultaneous video and audio flows, all as multicast, needs a mechanism for ensuring those streams only reach the switches and devices that actually need them. Without that mechanism, every multicast packet floods to every port on the network - which in a media facility would be catastrophic.

IGMP is that mechanism. Understanding how it works, and how it applies to a spine-leaf broadcast network, is practical knowledge for any engineer working with IP media infrastructure.

## What Multicast Is

Before IGMP, it helps to be clear on what IP multicast is and why it matters.

A **unicast** stream has one source and one destination. To send the same audio to ten receivers, you need ten separate streams. At 80 Mbps per 64-channel AES67 stream, that is 800 Mbps from a single source to serve ten destinations - and the source has to generate all ten copies.

A **multicast** stream has one source and potentially many receivers. The source sends one stream to a multicast group address (in the 224.0.0.0/4 range), and the network delivers copies only to the switches and devices that have requested them. The source generates one stream regardless of how many receivers are listening. This is how ST 2110 scales to the channel counts a broadcast facility requires.

The trade-off is complexity. Unicast is handled by normal switch forwarding tables. Multicast requires additional protocols to track who wants what, and to make sure streams are forwarded efficiently rather than flooded across the network.

## What IGMP Is

**IGMP (Internet Group Management Protocol)** is the protocol that hosts use to tell their local network switch or router which multicast groups they want to receive. When a device wants to join a multicast stream, it sends an IGMP Join message for that group address. When it no longer needs the stream, it sends a Leave message.

IGMP operates between a host (an AES67 receiver, a video decoder, a monitoring tool) and the first-hop switch or router on its local segment. It is a layer 3 protocol, carried directly in IP packets.

There are three versions in use:

**IGMPv1** - the original. Hosts can join groups but there is no explicit leave mechanism. A host simply stops responding to queries and the router times it out. This makes stream teardown slow and is not suitable for broadcast use.

**IGMPv2** - adds explicit Leave Group messages, which dramatically speeds up stream teardown. This is the minimum version you should find on any broadcast network. Most AES67 and ST 2110 equipment supports v2 as a baseline.

**IGMPv3** - adds **source-specific multicast (SSM)**. Rather than just joining a group address, a host can specify which source it wants to receive from: "I want multicast group 239.1.1.1, but only from source 10.0.0.5." This is significant for ST 2110, where multiple senders might use the same group address, and where the receiver needs to be explicit about which source it is joining. SMPTE ST 2110 recommends IGMPv3 for this reason.

## IGMP Snooping

IGMP is a host-to-router protocol. The switch in between does not natively understand it - left to its own devices, a Layer 2 switch treats multicast like broadcast and floods it to every port.

**IGMP snooping** is the switch feature that changes this. A snooping-enabled switch listens to the IGMP messages passing through it and builds a table of which ports have active receivers for which multicast groups. When a multicast packet arrives, the switch forwards it only to the ports in that group's table rather than flooding it everywhere.

This is not a router function - it happens at Layer 2, on the switch itself, transparently to the hosts. It is the first and most important line of defence against multicast flooding in a media network.

IGMP snooping should be enabled on every switch in a broadcast media network. On most enterprise-grade switches it is enabled by default, but it is worth verifying - particularly on any switch that is new to the network or that has been reset to factory defaults.

## The IGMP Querier

IGMP relies on a **querier** - a device that periodically sends IGMP Membership Query messages to the network, prompting all hosts to report which groups they are members of. This keeps the switch's multicast forwarding table current. Without queries, the table would eventually expire and multicast traffic would stop being forwarded correctly.

Normally the querier role is filled by a multicast router (running PIM - more on this below). In a network segment with no multicast router, or where the router is not participating in IGMP, a switch with IGMP snooping enabled can act as a querier instead. This is called **IGMP snooping querier** and needs to be explicitly configured on one switch per VLAN.

In a spine-leaf media network, the typical design places the querier function on the leaf switch, either through PIM on the switch's layer 3 interface or through the snooping querier feature if the leaf is operating purely at Layer 2. There should be exactly one active querier per VLAN - multiple queriers can interfere with each other, though the protocol does include an election mechanism to handle this.

## How It Applies to a Spine-Leaf Topology

In a spine-leaf broadcast network, multicast traffic flows according to a clear pattern that IGMP and its companion protocol PIM manage together.

### On the Leaf Switches

Leaf switches are where devices connect. IGMP snooping on each leaf ensures that a multicast stream arriving at the leaf is only forwarded to the specific ports where receivers have joined that group. If only one port on a leaf has a receiver for a given 239.x.x.x address, only that port gets the traffic - the other 47 ports on that leaf switch are not burdened with it.

When an AES67 receiver powers up and joins a stream, it sends an IGMPv2 or IGMPv3 Join to its local leaf. The leaf records the port in its snooping table. When the same stream is already arriving at the leaf for another receiver, the leaf simply adds the new port to the forwarding list for that group - no additional traffic needs to come from upstream.

When a receiver sends a Leave message, the leaf switch needs to determine whether any other ports on that leaf still want the stream. IGMPv2 handles this with a **Group-Specific Query** - the switch sends a query for that group and waits for responses. If no other receiver responds within the timeout (by default, a few seconds), the stream is pruned from that leaf. IGMPv3 handles this more efficiently for source-specific joins.

### Between Leaf and Spine

Traffic between different leaves is handled at Layer 3, which means IP routing and **PIM (Protocol Independent Multicast)** rather than IGMP. PIM is the routing protocol that builds the multicast distribution tree across the network - it handles which spine switches carry which streams between which leaves.

IGMP and PIM work together at the boundary. When a leaf sees a Join from a receiver for a group that is not yet arriving from upstream, it signals to the multicast routing layer (via PIM) that it needs that stream. PIM then arranges for the stream to be forwarded down to that leaf. When the last receiver on a leaf leaves a group, PIM prunes the stream from the path between the upstream source and that leaf.

The two common PIM modes used in broadcast facilities are:

**PIM Sparse Mode (PIM-SM)** - uses a Rendezvous Point (RP), a central router that acts as a meeting point for sources and receivers. Receivers join the RP-rooted tree initially, then can switch to a source-specific tree. Most deployments use PIM-SM with an RP configured on the spine layer or a dedicated multicast router.

**PIM Source-Specific Multicast (PIM-SSM)** - works directly with IGMPv3 source-specific joins. No RP is needed. Receivers specify the source explicitly, and PIM builds a direct tree from source to receiver. This is cleaner and is the recommended approach for ST 2110 deployments, provided all equipment supports IGMPv3.

### Join Latency

One operational characteristic that matters for broadcast is **join latency** - the time between a receiver sending an IGMP Join and the stream actually arriving at that port. In a well-configured spine-leaf network this is typically under a second, but several factors can extend it:

- The querier interval determines how quickly the network responds to membership changes. Shorter query intervals (configurable on the switch) reduce join and leave latency at the cost of slightly more IGMP traffic on the network.
- PIM convergence time adds to the delay when a stream needs to be pulled across a spine link for the first time.
- **Immediate-leave** (or fast-leave) is a switch feature that prunes a port immediately on receiving a Leave message without waiting for the Group-Specific Query. This speeds up stream teardown significantly and should be enabled on ports with a single receiver. It should not be enabled on ports where multiple receivers share the same switch port (for example, via a downstream unmanaged switch).

For audio routing applications - console snapshot recalls, routing matrix changes, live production switching - join latency is a real consideration. Dante and other AoIP protocols manage this through their own discovery and routing layers, but the underlying IGMP join still has to complete before audio flows. Understanding the network's join latency characteristics is important for commissioning and troubleshooting.

## Common Configuration Points

**Enable IGMP snooping on all VLANs used for media traffic.** Verify it is active, not just present in the configuration. Some platforms require it to be enabled globally and per-VLAN.

**Configure an IGMP querier on each media VLAN.** If there is no multicast router acting as querier, enable the snooping querier feature on one leaf switch per VLAN and assign it an IP address within that VLAN's subnet.

**Use IGMPv3 where possible.** ST 2110 with source-specific multicast benefits from v3, and most current broadcast equipment supports it. Where older devices support only v2, the network will fall back gracefully, but the source-filtering capability is lost.

**Set the IGMP version to match your equipment.** If you have mixed v2 and v3 devices, configure the switches to operate in the mode your devices expect and verify interoperability.

**Enable immediate-leave on access ports.** On ports connecting directly to single receivers (audio endpoints, video decoders), immediate-leave speeds up stream teardown when routing changes occur.

**Validate with traffic analysis.** After configuring IGMP snooping, verify with a port mirror or switch statistics that multicast is not flooding to ports that have no active receivers. In a media network, unexpected multicast traffic on a port is a clear sign that snooping is not working as expected.

## The Practical Picture

For a broadcast engineer, IGMP snooping is the switch feature that makes IP multicast practical at scale. Without it, a spine-leaf ST 2110 network with dozens of active streams would be flooding multi-gigabit multicast traffic to every connected device, overwhelming endpoints and wasting bandwidth across the fabric.

With it, each device sees only the streams it has asked for. The network carries traffic efficiently, sources generate one stream regardless of receiver count, and the routing layer handles distribution across the spine. Join and leave behaviour is controlled and fast enough for live production use.

The details of PIM configuration, RP placement, and IGMPv3 source filtering go deeper than this post covers - but the foundation is IGMP, and getting that right at the leaf layer is the first step to a multicast network that behaves correctly under broadcast operational conditions.
