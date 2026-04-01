---
date: 2026-04-01T00:00:00Z
draft: false
categories:
- Broadcast Technology
- AoIP
tags:
- ST 2110
- AES67
- IP
- Audio Engineering
- Redundancy
title: "SMPTE ST 2022-7 - Seamless Redundancy for IP Media"
description: "ST 2022-7 defines a hitless redundancy scheme for IP media streams, sending identical content over two independent network paths and silently discarding any packets that arrive late or not at all. Here is what it does, how it works, and where it fits in broadcast AoIP."
---

Broadcast infrastructure has always been built around redundancy. Dual power supplies, redundant signal paths, failover routing - the principle is the same everywhere: no single point of failure should take a show off air. When broadcast moved to IP, the question became how to achieve the same resilience on a packet network, where the failure modes are fundamentally different from SDI.

SMPTE ST 2022-7 is the answer the industry settled on. It defines a scheme called Seamless Protection Switching, and understanding it is worthwhile for any engineer working with AES67, ST 2110, or IP contribution systems at a level above basic connectivity.

## What ST 2022-7 Does

The core idea is straightforward. Instead of sending one copy of a media stream over one network path, you send two identical copies over two independent network paths simultaneously. At the receiving end, a device implementing ST 2022-7 monitors both streams, compares them packet by packet using RTP sequence numbers and timestamps, and reconstructs a single output stream by taking packets from whichever path delivers them first. If a packet is missing or late on one path, the corresponding packet from the other path is used instead. The failure is absorbed silently, with no interruption to the output.

This is what "seamless" and "hitless" mean in this context. There is no switching event, no momentary dropout, no audible click. From the perspective of anything downstream, the stream is continuous and complete, provided that no single packet is lost on both paths simultaneously.

## How It Works in Practice

The two network paths - usually called the Red network and the Blue network - need to be genuinely independent. That means separate switches, separate cabling, ideally separate physical routes. Running both paths through the same core switch defeats the purpose, because a failure in that switch takes out both paths at once.

At the receiver, the ST 2022-7 implementation maintains a small buffer - the **delay differential buffer** - which accommodates the difference in arrival time between the two paths. Packets from both paths are held briefly and compared. The buffer size needs to be large enough to cover the maximum expected delay difference between the paths, which in a well-designed facility network is typically a few milliseconds. If the paths are very different in length or traverse different numbers of hops, the buffer requirement increases accordingly.

The RTP sequence number is the key to the merging logic. Each packet in an RTP stream carries a sequence number that increments with every packet. The receiver uses these to identify duplicate packets arriving on the two paths and to detect gaps - packets that are present on one path but missing on the other. A missing packet on one path that is present on the other is silently substituted. A packet missing on both paths is a genuine loss event, and how that is handled depends on the receiving application.

## Advantages

**True hitless redundancy.** A network failure on one path - a switch reboot, a cable pulled, a port flapping - produces no audible output on a correctly implemented ST 2022-7 system. For live broadcast this is significant. The equivalent on an SDI infrastructure would be an automatic protection switch with milliseconds of dropout. ST 2022-7 produces zero dropout.

**Transparent to the application.** The redundancy is handled at the network/transport layer. The audio application receiving the stream does not need to know anything about the dual-path architecture. It sees a single, continuous RTP stream.

**Widely supported.** ST 2022-7 is part of the SMPTE ST 2110 suite and is broadly implemented across professional AoIP and video-over-IP hardware. Any system claiming ST 2110 compliance for production environments is expected to support it.

**Works alongside existing IP infrastructure.** You do not need specialised hardware to run the media streams themselves - standard managed switches are fine, provided you have two independent paths. The complexity sits in the sender and receiver implementations, not the network.

## Disadvantages

**Doubles the network bandwidth requirement.** Two identical streams means twice the bandwidth consumed for every flow. On a media network with high channel counts this adds up quickly. A 64-channel AES67 stream at 1ms packet time already consumes meaningful bandwidth; running two copies of every stream on separate VLANs or physical networks requires careful capacity planning.

**Requires genuinely independent infrastructure.** Two independent switch fabrics, two sets of cabling, two network paths from every source to every destination. For a permanent broadcast centre this is achievable but expensive. For a temporary production - an OB truck or a temporary venue installation - the space, cost, and complexity of duplicating the network infrastructure is a significant overhead.

**Buffer latency.** The delay differential buffer adds latency. On a well-designed network with short, similar paths this is small - typically under 10ms. On networks where path asymmetry is large, or where the two paths traverse very different amounts of infrastructure, the buffer requirement and therefore the added latency grows. For most audio contribution applications this is acceptable, but it is a consideration in low-latency monitoring workflows.

**Failure is silent.** This is a double-edged characteristic. Hitless operation means a path failure produces no audible output, which is exactly what you want. But it also means a path failure can go unnoticed if monitoring is not in place. A system running on one path after a silent failure has lost its redundancy, and a second failure will cause a dropout. Monitoring the health of both paths independently is essential - the redundancy only has value if you know when it has been consumed.

## Applications in AoIP

In a SMPTE ST 2110 facility, ST 2022-7 is standard practice for any signal that matters. The ST 2110-30 audio standard is implemented alongside ST 2022-7 in essentially every serious deployment. Senders transmit on both Red and Blue networks simultaneously; receivers merge the two streams. The audio infrastructure inherits the same redundancy model as the video.

For AES67 systems outside a full ST 2110 environment, ST 2022-7 support is less universal but increasingly common in professional hardware. Devices from manufacturers like Lawo, Merging Technologies, and Grass Valley implement it for their AoIP interfaces. Whether a specific device supports ST 2022-7 is worth confirming during system design rather than assuming.

In contribution and remote production workflows, where IP circuits are traversing wide area networks or third-party infrastructure, ST 2022-7 can be applied to contribution streams provided both paths are available. In practice, WAN contributions more commonly use application-layer redundancy or forward error correction instead, because running two independent WAN circuits to the same location is expensive. ST 2022-7 is primarily a facility and campus network tool.

## A Practical Note on Network Design

The value of ST 2022-7 is entirely dependent on the independence of the two paths. I have seen installations where Red and Blue networks share uplinks, share core switches, or share physical cable routes through the same duct - which means a single failure can affect both paths simultaneously and the seamless protection provides no real benefit.

When commissioning an ST 2022-7 system, it is worth testing failure scenarios explicitly: pull a cable, disable a switch port, reboot a switch, and verify that the output is uninterrupted and that monitoring correctly identifies the path failure. If you cannot demonstrate a clean single-path failure test, the redundancy is not working as intended.

The monitoring piece is equally important. ST 2022-7 changes the consequence of a network failure from an immediate audible event to a silent degradation of redundancy. That is a better failure mode, but only if the operations team knows about it and can act before a second failure occurs.
