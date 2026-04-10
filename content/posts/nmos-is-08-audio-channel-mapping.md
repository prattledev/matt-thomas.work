---
date: 2026-04-06T12:00:00Z
draft: false
categories:
  - Broadcast Technology
  - AoIP
tags:
  - NMOS
  - AoIP
  - IP
  - Audio Engineering
  - IS-08
  - ST 2110
title: "NMOS IS-08: Audio Channel Mapping"
description: "IS-08 is the NMOS specification for controlling how audio channels within a network flow are assigned to physical outputs on a receiver. It solves a practical problem that IS-05 alone cannot: what happens inside the flow once the connection is made."
cover:
  image: nmos.png
  alt: "NMOS IS-08 Audio Channel Mapping"
---

IS-05 handles making the connection between a sender and a receiver. Once that connection exists, a multi-channel audio flow is arriving at the receiver - but which channels end up on which outputs is a separate question, and IS-05 has nothing to say about it.

That is the gap IS-08 fills. It is the NMOS specification for audio channel mapping: a standardised API for controlling how the audio channels within a received flow are routed to the physical or logical output pins of a device.

## The Problem IS-08 Solves

In an SDI or MADI world, channel assignment is determined at the point where the signal is created. If a console expects its primary input on channels 1 and 2 of a MADI feed, those channels have to be placed there before the signal leaves the source device. Changing the assignment means going back to the source.

In an IP audio world, flows frequently carry many more channels than any individual destination needs. A 64-channel ST 2110-30 flow might be the output of a stage box, carrying every input on that box across the network to wherever it is needed. Different destinations - a monitor mix console, a broadcast mix console, a recording system - all connect to the same flow but need different subsets of channels, potentially in different orders.

Without IS-08, a receiver presents whatever channel order is in the flow, and adjusting the assignment requires either reconfiguring the source or relying on whatever proprietary interface the receiving device offers. IS-08 gives control systems a standardised way to manage this on the receiver side, independently of the source.

## How IS-08 Works

IS-08 models the receiver as having two layers: **inputs** and **outputs**.

**Inputs** represent the audio channels arriving from the network. If the device has an active IS-05 connection receiving a 64-channel flow, there are 64 inputs, corresponding to channels 1 through 64 of that flow. If there is no active connection, the inputs have no signal.

**Outputs** represent the physical or logical output pins of the device - the AES3 outputs, the MADI channels, the console fader inputs, the monitor feed. These are fixed by the hardware; their count and labelling are defined by the device manufacturer.

The **channel map** is the matrix that connects inputs to outputs. Each output is assigned to one input. You can assign any input to any output, assign the same input to multiple outputs (duplicating a channel), or leave an output unrouted (silence). You cannot, within the IS-08 model, mix or blend channels - it is a routing matrix, not a mixer.

The API exposes two maps: a **staged** map and an **active** map. The staged map is where you write a new assignment before committing it. The active map is what the device is currently using. Activation works the same way as IS-05 - immediately, or at a specified PTP timestamp - so a channel remap can be coordinated with a connection change to happen at the same instant.

## A Practical Example

A host outside broadcast truck is receiving a 16-channel ST 2110-30 flow from a stage box. The stage box channels are not in the order the console engineer wants - the pitch fx microphones are on channels 9-16, but the console template has them patched to inputs 1-8.

With IS-08, a control system can remap the received flow so that stage box channels 9-16 appear on the console's first eight inputs, without touching anything at the stage box or reconfiguring the flow itself. The change is made on the receiver, takes effect at the next appropriate PTP boundary, and the source is unaware anything changed.

The same flow, routed simultaneously to another rights holder broadcast truck, can have a completely different channel map configured on that receiver - the sound engineer's preferred layout, independent of what the host broadcast console sees.

So, in old school broadast terms, each receiver has its own audio shuffler at its disposal. A powerful feature.

## Inputs Without a Connection

IS-08 defines the concept of **routable inputs** and **unroutable inputs**. A routable input is one that a control system is permitted to assign. An unroutable input might be a hardwired channel that the device manufacturer has fixed - a reference tone or a device-specific signal that should not be overridden by external control.

When there is no active IS-05 connection on a receiver, the inputs associated with that connection carry no signal. Outputs mapped to those inputs will be silent. This is expected behaviour - the channel map persists regardless of whether there is an active flow, and when a connection is established the mapped channels become available without any further configuration.

## Relationship to IS-04 and IS-05

IS-08 sits on top of IS-04 and IS-05 rather than replacing any part of them. The workflow in a system that supports all three looks like this:

1. IS-04 discovers that a sender and a compatible receiver exist on the network.
2. IS-05 establishes the connection - the flow starts arriving at the receiver.
3. IS-08 configures how the channels within that flow are assigned to the receiver's outputs.

Steps 2 and 3 can be coordinated to activate at the same PTP time, so the connection and the channel assignment take effect simultaneously. A media orchestration platform managing all three specs can present this as a single routing operation to the operator, hiding the underlying sequence.

## Adoption and Limitations

IS-08 is less uniformly adopted than IS-04 and IS-05. Many devices that implement the core NMOS specifications have not yet added IS-08 support, particularly at the lower end of the market. It is worth checking device specifications explicitly if channel mapping control is a requirement - NMOS compliance alone does not guarantee IS-08 is implemented.

The matrix model also has limits. IS-08 handles routing within a received flow, but it has no mechanism for format conversion, sample rate conversion, or mixing. If a flow arrives at a sample rate or bit depth the receiver cannot handle, IS-08 cannot compensate - that is a problem for IS-11 (Stream Compatibility Management) and the capabilities negotiation it provides.

For devices that do implement IS-08, the depth of implementation varies. Some expose the full matrix with per-output assignment. Others implement a simplified version that only supports contiguous channel blocks. The AMWA specification allows for this variability, so testing against specific hardware is important before designing a workflow that depends on fine-grained channel control.

## Why It Matters for AoIP

For audio engineers working in IP facilities, IS-08 is the specification that makes large multi-channel flows practically useful. Without it, the flexibility of sending a 64-channel flow to multiple destinations is undermined by the need to negotiate channel assignment through proprietary interfaces or to pre-configure the source for each destination's requirements.

With it, channel assignment becomes part of the routing layer - something a control system can manage consistently, log, and change as cleanly as an IS-05 connection. In a facility running a media orchestration platform that supports IS-08, a routing change that would previously have required communication between the source operator, the destination operator, and whoever manages the network can become a single operation with an audit trail.

The practical implication for any new AoIP deployment is to include IS-08 support in the device specification checklist alongside IS-04 and IS-05, particularly for receivers that will be handling multi-channel flows where the channel assignment needs to be flexible or production-specific.
