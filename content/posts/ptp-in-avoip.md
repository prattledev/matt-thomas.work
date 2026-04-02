---
date: 2026-03-29T18:00:00Z
draft: false
categories:
- Broadcast Technology
- AoIP
tags:
- PTP
- AES67
- ST 2110
- Audio Engineering
- IP
title: 'PTP in AVoIP - A Practical Guide for Audio Engineers'
description: "PTP is the invisible foundation that makes synchronised audio over IP possible. Understanding how it works - and how to troubleshoot it - is essential for anyone working with AES67 and ST 2110 systems."
cover:
  image: ptp2110.png
  alt: "PTP grandmaster, transparent clocks and boundary clocks in a spine-leaf ST 2110 network"
---

Of all the things that can go wrong in an AoIP system, PTP problems are among the most frustrating to diagnose. The audio often still plays - just with intermittent glitches, drift, or lip sync issues that are hard to reproduce and harder to pin down. Understanding what PTP is doing, and why, makes a significant difference when you're standing in a broadcast centre an hour before air wondering why your streams are misbehaving.

## What PTP Is and Why AoIP Needs It

PTP - Precision Time Protocol, defined in IEEE 1588 - synchronises clocks across a network to sub-microsecond accuracy. In an AoIP system, every device that sends or receives audio needs to agree on what time it is. That shared time reference is how RTP timestamps work - each audio packet carries a timestamp that tells the receiver exactly when that sample was captured, allowing it to reconstruct the audio stream correctly and align it with streams from other sources.

Without a common time reference, audio from different sources will drift relative to each other. On a small system you might get away with it briefly, but any serious multi-source production environment requires PTP.

AES67 mandates IEEE 1588-2008 (PTPv2). SMPTE ST 2110-30, which governs audio in an ST 2110 environment, inherits the same requirement. The clock reference is also embedded in SDP files via the `ts-refclk` attribute - which is how receivers verify they share the same grandmaster as the sender before accepting a stream.

## PTP Versions

**IEEE 1588-2002 (PTPv1)** - the original specification. Rarely seen in broadcast environments today, though notably Dante uses PTPv1 for its internal clock synchronisation. It is incompatible with v2 at the message level, which is worth keeping in mind when Dante devices share a network with AES67 equipment.

**IEEE 1588-2008 (PTPv2)** - the version you'll encounter in virtually every AoIP system. Significantly improved accuracy over v1, introduced the concept of transparent clocks, and added better support for different network topologies. This is what AES67 and ST 2110 use. Dante also switches to PTPv2 when operating in AES67 mode, which is how it can coexist on the same clock domain as other AES67 devices.

**IEEE 1588-2019 (PTPv3)** - the current revision. Adds security features (authentication of PTP messages) and some architectural improvements. Adoption in broadcast is still limited, but it is backward-compatible with v2 in many respects.

**IEEE 802.1AS (gPTP)** - a profile of IEEE 1588 used in AVB and TSN networks. Tighter constraints than standard PTP, requires hardware support throughout the network. You'll encounter this in Dante Domain Manager environments and TSN-based systems rather than in traditional AES67 deployments.

## PTP Profiles

A PTP profile defines a specific set of parameters and constraints on top of the base standard - things like multicast vs unicast operation, message intervals, and which clock selection algorithm to use.

**AES67 profile** - uses multicast PTP, follows IEEE 1588-2008 default profile conventions. Most standalone AES67 devices follow this.

**SMPTE ST 2059-2** - a broadcast-specific PTP profile that adds epoch and frame rate alignment on top of PTPv2. Designed to lock PTP time to a video reference, making it possible to align audio timestamps with video frames. If you're working in a full ST 2110 environment, this is the profile you're likely using.

Understanding which profile your devices are running matters when mixing equipment from different manufacturers. A device running the ST 2059-2 profile and a device running the default AES67 profile may not interoperate cleanly as master and slave, even though both speak PTPv2.

## Clock Types

**Grandmaster Clock (GM)** - the top of the hierarchy. Every PTP domain has one active grandmaster, and all other devices ultimately derive their time from it. In a broadcast facility, the grandmaster is typically locked to GPS or an atomic clock reference. In a temporary production environment, one of the devices on the network will assume the grandmaster role through the Best Master Clock Algorithm (BMCA).

**Ordinary Clock (OC)** - the most common type. A device with a single PTP port that operates as either a master or slave. Most AoIP endpoints are ordinary clocks.

**Boundary Clock (BC)** - a device with multiple PTP ports that terminates PTP on one side and re-generates it on the other. A boundary clock in a managed switch means each network segment has its own local master, reducing the number of devices competing for the grandmaster and improving accuracy across the network. In larger AoIP deployments, boundary clocks in the network infrastructure are important for maintaining tight synchronisation.

**Transparent Clock (TC)** - passes PTP messages through without terminating the protocol, but corrects for the time the message spends inside the device (residence time). This prevents switch-induced delay from corrupting the offset calculation. End-to-end transparent clocks are common in AoIP-aware network switches. Without transparent clocks or boundary clocks in your switches, PTP accuracy degrades significantly as network scale increases.

## The Best Master Clock Algorithm

When a PTP domain starts up, devices use the BMCA to elect a grandmaster. The algorithm compares devices in priority order:

1. Priority1 (manually configured, lower value wins)
2. Clock class (indicates clock quality - a GPS-locked clock will have a higher class)
3. Clock accuracy
4. Offset scaled log variance (a measure of clock stability)
5. Priority2 (secondary manual priority)
6. Clock identity (the device's EUI-64, used as a tiebreaker)

In a production environment, you typically want to explicitly set Priority1 on your designated grandmaster to ensure it always wins the election. Leaving everything at default and hoping the right device wins is a recipe for problems when a device reboots mid-show.

## PTP Domains

A PTP domain is a logical grouping of clocks that synchronise with each other. Devices only exchange PTP messages with other devices in the same domain - a device in domain 0 will completely ignore PTP traffic from a device in domain 127, even if they're on the same network segment.

Domain numbers run from 0 to 255. Domain 0 is the default and is what most AoIP devices use out of the box. SMPTE ST 2059-2 recommends domain 127 for the broadcast profile, but in practice audio and video devices typically share the same PTP domain - everyone locked to the same grandmaster, on the same domain number.

Running multiple domains on the same network is possible, but in most production environments a single domain is simpler and sufficient. Where separate domains are used, it's usually to isolate a redundant or backup clock infrastructure rather than to separate media types.

The practical implication is that when a device isn't locking to PTP, checking the domain number is one of the first things to verify. It's a surprisingly common misconfiguration - particularly when mixing equipment from manufacturers that ship with different domain defaults.

## Network Considerations

PTP accuracy is heavily influenced by the network it runs on. A few things that matter in practice:

**Hardware timestamping** - PTP timestamps should be applied as close to the physical layer as possible. Software timestamping, where the operating system applies the timestamp, introduces jitter that limits accuracy. Managed switches with hardware PTP support (transparent or boundary clocks) make a significant difference.

**Asymmetric delay** - PTP assumes the path from master to slave has the same delay as the path from slave to master. If the network introduces asymmetric latency (different delay in each direction), the offset calculation will be wrong. This can be an issue with certain switch configurations.

**Multicast vs unicast** - AES67 uses multicast PTP by default, which works well on a dedicated media network. On shared or routed networks, unicast PTP negotiation may be preferable to avoid flooding multicast traffic across the infrastructure.

## Tools

**[PTP Track Hound](https://www.ptptrackhound.com/#/home)** is a free Windows-based tool that passively listens to PTP traffic on your network and gives you a real-time view of what's happening - grandmaster identity, clock offsets, path delay, and which devices are acting as masters and slaves. It's one of the most useful diagnostic tools available for AoIP work and well worth having on your laptop.

**Wireshark** with PTP dissection enabled is useful for deeper analysis - you can inspect individual PTP messages and trace exactly what each device is sending and receiving.

**ptp4l** is the standard Linux PTP implementation and is available on most distributions. Useful for testing from a laptop or server, and for understanding PTP behaviour at a lower level.

Most professional AoIP devices also include some form of PTP status display - offset from master, grandmaster identity, and lock status. Getting familiar with what these look like on the specific equipment you work with is time well spent before you're under pressure on a live event.
