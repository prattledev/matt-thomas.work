---
date: 2026-03-29T14:00:00Z
draft: false
categories:
- Broadcast Technology
- AoIP
tags:
- NMOS
- AoIP
- IP
- Audio Engineering
- IS-04
- IS-05
title: 'What is NMOS, and Why Does It Matter for AoIP?'
description: "AoIP solves how audio travels across a network. NMOS tackles what comes next - how devices discover each other and how connections are made and managed."
---

One of the things I find interesting about the broadcast industry's move to IP is how the conversation tends to focus on transport. Ravenna, AES67, ST 2110 - these are protocols for moving media across a network, and they've matured significantly over the last decade. But transport is only part of the picture.

Once you have dozens or hundreds of IP devices on a network, you need answers to some fairly fundamental questions. How do applications know what devices are available? How do you make a connection between a sender and a receiver? How do you change that connection, and when does the change take effect?

In an SDI or MADI world, the answers are physical. You patch a cable. You can see what's connected. In an IP world, without a standard way to handle this, every vendor builds their own control plane - and interoperability falls apart at exactly the layer where you need it most.

That's what NMOS addresses.

## What NMOS is

NMOS - Networked Media Open Specifications - is a family of open standards developed by AMWA, the Advanced Media Workflow Association. The specifications cover different aspects of managing networked media infrastructure, from device discovery through to connection management, audio channel mapping, and event distribution.

Two specifications sit at the foundation: IS-04 and IS-05.

## IS-04: Discovery and Registration

IS-04 is how devices announce themselves and how applications find them. When an NMOS-compliant device comes online, it registers itself with a central registry - advertising what it is, what it can send, and what it can receive. Applications can then query that registry to get a real-time picture of what's available on the network.

The model breaks down into a few key concepts. A **Node** is a physical or virtual device. Each node contains **Devices** - logical groupings of functionality. Devices have **Senders** (things that generate media flows) and **Receivers** (things that consume them). **Sources** and **Flows** describe the content itself.

In practice, this means a control application can query the IS-04 registry and immediately know every sender and receiver on the network, what formats they support, and whether they're currently connected to anything. Without something like this, you're back to manually maintaining configuration files or relying on proprietary discovery mechanisms that don't work across vendors.

## IS-05: Connection Management

If IS-04 tells you what's available, IS-05 is how you actually connect things. It provides a standardised API for making, changing, and removing connections between senders and receivers - and critically, it's transport-independent. The same connection management workflow applies whether the underlying transport is RTP (as with Ravenna or AES67), WebSocket, or anything else.

One aspect of IS-05 I find particularly useful is the timing control. You can activate connections immediately, or schedule them to take effect at a specific PTP time. For live broadcast, being able to coordinate connection changes across multiple devices at a precise moment is genuinely valuable - it's the difference between a clean switch and a glitch.

IS-05 also explicitly fills a gap that ST 2110 left open. SMPTE defined how to transport media over IP, but deliberately said nothing about how connections should be established or managed. IS-05 is the answer to that question.

## Why this matters for AoIP

For anyone working with AoIP systems, NMOS represents the control plane that the transport protocols don't provide. Ravenna and AES67 define how audio moves across the network. NMOS defines how you manage the infrastructure around that.

The practical implication is interoperability. A Ravenna device from one manufacturer and an AES67 receiver from another can both register with the same IS-04 registry, and a control application can connect them via IS-05 without either vendor needing to implement proprietary integration. That's the promise, at least - and increasingly, it's the reality, particularly in larger broadcast facilities where NMOS adoption has been driven by broadcasters who simply won't accept vendor lock-in at the control layer.

Adoption is still uneven, particularly in live event and temporary broadcast centre contexts where I spend most of my time. A lot of AoIP deployments at that level still rely on proprietary control surfaces and manual configuration. But the direction is clear, and for anyone specifying new AoIP infrastructure, NMOS compatibility is worth treating as a baseline requirement rather than an optional feature.
