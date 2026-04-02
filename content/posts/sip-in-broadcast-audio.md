---
date: 2026-03-30T00:00:00Z
draft: false
categories:
  - Broadcast Technology
tags:
  - SIP
  - Audio Engineering
  - Comms
  - IP
  - Codecs
title: "SIP in Broadcast Audio - Commentary and Coordination Circuits Over IP"
description: "ISDN is gone and dedicated circuits are expensive. SIP gives broadcast facilities a standardised, interoperable way to bring commentary and coordination audio in and out over the internet."
cover:
  image: sip.png
  alt: "SIP architecture - remote codec connecting to broadcast centre via public internet"
---

For most of my career, getting a high-quality audio circuit to or from a remote location meant either booking an ISDN line or arranging a satellite or fibre contribution. ISDN was reliable, well-understood, and nearly universal in broadcast. It was also expensive, inflexible, and is now being switched off across much of the world.

The replacement, in most cases, is SIP over IP. And while IP contribution codecs have been around for a long time, SIP as a standardised signalling layer has made a real difference to interoperability - particularly as broadcast facilities want to bring commentary and coordination circuits in from the public internet without needing to manage dedicated infrastructure at the remote end.

## What SIP Is

SIP - Session Initiation Protocol - is defined in RFC 3261. It's a text-based signalling protocol responsible for establishing, modifying, and terminating multimedia sessions. SIP itself doesn't carry audio - that's handled by RTP, the same transport protocol used in AoIP systems like AES67. SIP is purely the call setup and teardown layer.

This separation of signalling and media is important. It means SIP can negotiate what codec to use, where to send the audio, and how to handle the session, without being tied to any specific audio format. The codec negotiation happens via SDP - the same Session Description Protocol used in ST 2110 systems.

## Why It Works for Broadcast Contribution

Commentary and coordination circuits have a few specific requirements that SIP handles well.

Remote locations are often temporary and unpredictable. A commentator at a stadium, a reporter in a field, a presenter connecting from home - none of these have guaranteed network infrastructure. SIP works over any IP connection, including 4G and 5G, which means circuits that would previously have required a satellite uplink or pre-booked ISDN can now be established from a laptop or a small hardware codec with a mobile data connection.

The signalling model also suits broadcast workflows. A SIP call is explicitly set up and torn down, which maps naturally to the concept of a contribution circuit. You know when it's active, you can monitor it, and you can re-establish it if it drops. Compare this to unmanaged RTP streams, where the sending device just starts transmitting and you hope the receiver is ready.

For coordination circuits - talkback to journalists, IFB to presenters, intercom between production staff at remote sites - SIP has become the default in a lot of facilities, particularly as intercom systems from manufacturers like Riedel, RTS, and Clear-Com have added SIP support to their gateways.

## Basic Architecture

A typical broadcast SIP setup for incoming contribution circuits looks like this:

**At the facility or in the cloud:**

A **Session Border Controller (SBC)** sits at the edge of the network, facing the internet. The SBC handles NAT traversal, authenticates incoming SIP connections, and acts as a security and interoperability layer between the public internet and the internal facility network. It also handles transcoding if the remote codec and the internal system don't share a common codec.

Behind the SBC, a **SIP registrar/proxy server** manages endpoint registration and call routing. In smaller facilities this might be a single server or even built into the SBC. In larger facilities it might be a dedicated platform.

The audio from established calls is presented to the **broadcast console or router** - either as an analogue or AES3 output from a gateway device, or as an AoIP stream if the facility runs an IP audio infrastructure.

**At the remote end:**

This can be almost anything - a hardware contribution codec, a laptop running a SIP softphone, a smartphone. For broadcast-quality commentary the remote end is typically a dedicated hardware codec (Tieline, Comrex, Luci, and similar) or a purpose-built software client. For coordination and talkback, a good quality softphone on a laptop or phone is often sufficient.

**NAT traversal:**

Getting SIP to work reliably across the internet requires handling NAT - the mechanism by which routers share a single public IP address across multiple devices. SIP has historically had problems with NAT because SDP embeds IP addresses directly in the message body, which NAT doesn't modify.

The standard solutions are STUN (which lets a device discover its own public IP), TURN (which relays media through a server when direct connection isn't possible), and ICE (which tries multiple connection methods and picks the best one). In practice, the SBC at the facility handles most of this complexity - the remote device just needs to be able to reach the SBC.

## Codecs

Codec selection determines audio quality, latency, and interoperability. SIP negotiates codecs via SDP offer/answer - the calling device lists what it supports, the receiving device picks from that list.

**G.711** is the baseline codec every SIP implementation supports. It's 64kbps, narrowband (8kHz), and sounds like a telephone call. It's not suitable for programme audio but is useful as a fallback for coordination where intelligibility matters more than quality.

**G.722** is a step up - wideband at 16kHz, still 64kbps. Considerably better for voice, and widely supported. A reasonable choice for talkback and coordination circuits where you need broad compatibility.

**Opus** is the codec I'd recommend for most broadcast contribution use today. It's open source, royalty-free, and covers an enormous range of bitrates - from 6kbps up to 510kbps - with a single encoder. Crucially, it handles variable network conditions well, which matters on public internet connections. At higher bitrates it delivers audio quality that is genuinely competitive with traditional contribution codecs. Support across SIP stacks and hardware codecs has grown significantly in recent years, and for new deployments it's increasingly the sensible default.

**AAC-LD and AAC-ELD** are used in many professional broadcast codecs and offer good quality at lower bitrates. The limitation is that AAC is less universally supported in generic SIP stacks, so interoperability outside the broadcast codec ecosystem can be awkward.

**G.722.1 (Siren)** appears in some broadcast-specific hardware but has limited support outside that world.

## Interoperability Between Vendors

SIP's biggest practical advantage over proprietary contribution codec protocols is that it provides a common signalling framework. A Tieline codec, a Comrex unit, a software client, and a facility SBC can all establish calls with each other without vendor-specific configuration - provided they share a common codec.

In reality, interoperability is best when both ends support Opus or G.722 as a minimum. G.711 is the universal fallback but the quality cost is significant for anything other than coordination audio. Most modern hardware broadcast codecs support Opus, and most SIP softphones do too, so in practice a well-configured SBC with sensible codec priority ordering handles most situations automatically.

## A Practical Note on Latency

IP contribution over the public internet introduces variable latency that ISDN didn't have. For commentary circuits this is usually manageable - a few hundred milliseconds of total round-trip latency is acceptable for most workflows. For IFB and talkback, latency becomes more noticeable and more disruptive. Keeping the SBC geographically close to the remote location, using codecs with low algorithmic delay (Opus has a mode specifically for this), and using direct media paths where possible all help. But it's worth setting realistic expectations - internet contribution circuits will never have the determinism of a dedicated line.
