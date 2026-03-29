---
date: 2026-03-29T16:00:00Z
draft: false
categories:
- Broadcast Technology
- AoIP
tags:
- ST 2110
- AES67
- IP
- Audio Engineering
- SDP
title: 'SDP Files in ST 2110 - What They Are and How They Work'
description: "SDP files are how IP audio and video streams describe themselves. Understanding their structure is essential for working with AES67 and ST 2110 systems."
---

If you've spent any time working with ST 2110 or AES67 systems, you've almost certainly encountered SDP files. They show up everywhere - in NMOS sender manifests, in device configuration interfaces, in Wireshark captures, and in the logs of things that aren't working the way you expected.

They look deceptively simple. They're plain text. But getting them wrong causes problems that can be frustrating to diagnose, especially when the issue is something subtle like a mismatched packet time or an incorrect clock reference.

## What SDP Is

SDP stands for Session Description Protocol. It's defined in [RFC 4566](https://datatracker.ietf.org/doc/html/rfc4566) and is a standardised text format for describing multimedia sessions - specifically the parameters needed for a receiver to decode and synchronise an RTP stream.

SDP is not a transport protocol. It doesn't carry audio. It's purely descriptive - a structured way of communicating "here is a stream, this is what it contains, this is where it lives, and this is how you should handle it."

In the context of ST 2110, SDP is how senders advertise their streams and how receivers know how to join and decode them. In an NMOS environment, the IS-04 sender resource includes a `manifest_href` - a URL pointing to the sender's SDP file. IS-05 uses SDP parameters to configure connections between senders and receivers. I wrote more about NMOS and how IS-04 and IS-05 work in [a separate post](/posts/what-is-nmos/).

## SDP File Structure

SDP files are line-based, with each line following the format `type=value`. Lines are grouped into a session-level section followed by one or more media-level sections.

### Session-level lines

```
v=0
```
Version - always 0. Never changes.

```
o=- 1703123456 1703123456 IN IP4 192.168.1.10
```
Origin. Fields are: username (`-` if unused), session ID, session version, network type (`IN` for internet), address type (`IP4` or `IP6`), and the sender's unicast source address. The session ID and version are typically Unix timestamps. If the stream configuration changes, the session version increments.

```
s=My AES67 Stream
```
Session name - a human-readable label for the stream.

```
t=0 0
```
Timing. Start time and stop time in NTP seconds. `0 0` means the session is permanent.

### Media-level lines

```
m=audio 5004 RTP/AVP 98
```
Media description. Fields are: media type (`audio`), destination port, transport protocol (`RTP/AVP`), and payload type number. Payload type 98 is a dynamic type - its format is defined in the `rtpmap` attribute below.

```
c=IN IP4 239.69.0.1/32
```
Connection data - the multicast group address the stream is sent to. The `/32` is the multicast TTL.

```
a=rtpmap:98 L24/48000/2
```
Maps the dynamic payload type to a codec. Here: payload 98 is 24-bit linear PCM (`L24`), 48kHz sample rate, 2 channels.

```
a=fmtp:98 channel-order=SMPTE2110.(ST)
```
Format parameters. The `channel-order` attribute follows the SMPTE ST 2110-30 channel grouping convention. `ST` means standard stereo pair.

```
a=ptime:1
```
Packet time in milliseconds - how many milliseconds of audio are carried in each RTP packet. 1ms is the standard AES67 value, though 0.125ms is also common for low-latency applications.

```
a=mediaclk:direct=0
```
Specifies the media clock relationship. `direct=0` means the RTP timestamps map directly to the PTP clock with no offset.

```
a=ts-refclk:ptp=IEEE1588-2008:00-0C-E7-FF-FE-00-00-01:0
```
Defines the PTP grandmaster clock the sender is locked to, identified by its EUI-64 address. The `:0` at the end is the PTP domain number. This is critical for synchronisation - receivers use this to verify they share the same clock reference as the sender.

---

## A Typical AES67 Sender SDP

Stereo, 24-bit, 48kHz, 1ms packet time:

```
v=0
o=- 1703123456 1703123456 IN IP4 192.168.1.10
s=Studio A Mix L/R
t=0 0
m=audio 5004 RTP/AVP 98
c=IN IP4 239.69.0.1/32
a=source-filter: incl IN IP4 239.69.0.1 192.168.1.10
a=rtpmap:98 L24/48000/2
a=fmtp:98 channel-order=SMPTE2110.(ST)
a=ptime:1
a=mediaclk:direct=0
a=ts-refclk:ptp=IEEE1588-2008:00-0C-E7-FF-FE-00-00-01:0
a=sendonly
```

The `source-filter` line uses Source Specific Multicast (SSM) to tie the multicast group to the specific sender address - important in larger networks to prevent loops and ensure receivers join the correct source. `a=sendonly` makes the stream direction explicit.

---

## A Typical AES67 Receiver SDP

A receiver's SDP describes what it is subscribed to. It mirrors the sender's SDP with the direction reversed:

```
v=0
o=- 1703123456 1703123456 IN IP4 192.168.1.20
s=Studio A Mix L/R
t=0 0
m=audio 5004 RTP/AVP 98
c=IN IP4 239.69.0.1/32
a=source-filter: incl IN IP4 239.69.0.1 192.168.1.10
a=rtpmap:98 L24/48000/2
a=fmtp:98 channel-order=SMPTE2110.(ST)
a=ptime:1
a=mediaclk:direct=0
a=ts-refclk:ptp=IEEE1588-2008:00-0C-E7-FF-FE-00-00-01:0
a=recvonly
```

The receiver's own unicast address appears in the `o=` line (`192.168.1.20`), but the multicast group, port, payload format, and clock reference are identical to the sender. The direction is `recvonly`. Everything else must match - a mismatch in `ptime`, sample rate, or clock domain is a common source of audio dropouts or failed connections.

---

## Why This Matters in Practice

The SDP file is the contract between a sender and receiver. In an NMOS-managed system, IS-05 uses the SDP parameters to configure connections automatically. But in many live event environments, SDP files still get exchanged manually - copied from a device's web interface and pasted into a receiver's configuration.

In those situations, understanding what each line does makes the difference between diagnosing a problem in seconds and spending an hour in a matrix metering page wondering why the audio sounds wrong.
