---
date: 2026-03-29T12:00:00Z
draft: false
categories:
  - Broadcast Technology
  - AoIP
tags:
  - MXL
  - AoIP
  - IP
  - Audio Engineering
  - Cloud
title: "What is MXL, and Why Does It Matter for Broadcast Audio?"
description: "Most AoIP discussions focus on network transport. MXL tackles a different problem — how audio moves between software processes in cloud and containerised broadcast environments."
---

When we talk about audio over IP in broadcast, the conversation almost always centres on network transport. How do we get audio from A to B over an IP network? That's where Ravenna, AES67, Dante, and MADI-over-IP live. It's an important problem, and the industry has largely solved it.

But there's a different problem that gets less attention: once audio arrives inside a software-defined or cloud-based facility, how does it move efficiently between applications? How does one process hand audio to the next without creating a bottleneck?

That's the problem MXL is designed to solve.

## What MXL actually is

MXL - the Media eXchange Layer - is an open-source SDK that implements the EBU's Dynamic Media Facility reference architecture. It's designed for sharing media between software processes running in the same environment, whether that's a cloud instance, a containerised deployment, or an on-premise software stack.

The key concept is zero-copy shared memory. Rather than each application in a chain copying audio data as it passes it along, MXL allows processes to share access to the same memory. For high channel counts and low-latency requirements, this makes a meaningful difference - copying large amounts of audio data repeatedly is expensive, and in a software-defined facility running many concurrent processes, that cost adds up quickly.

It supports audio (Float32), video, and ancillary data, and it's built to be container-friendly from the ground up - Docker and Kubernetes deployments are explicitly supported.

## Why this matters for AoIP

AoIP protocols like Ravenna and AES67 are excellent at moving audio across a network. What they don't address is what happens once that audio lands inside a software environment and needs to flow through a chain of processing applications.

In a traditional hardware facility, this isn't really a concern - audio moves between physical devices over IP, MADI, AES3 or analogue. And each device handles its own processing. But in a cloud or software-defined facility, where everything is a container or a process, the equivalent of that physical interconnect is shared memory. If each handoff between processes involves a full copy of the audio buffer, you're introducing latency and CPU overhead that compounds quickly at scale.

MXL provides a standardised, open, and non-proprietary way to handle those handoffs efficiently. That's significant - without something like this, every vendor ends up implementing their own approach, and interoperability suffers.

## Where it fits

I'd think of MXL as operating at a different layer to AoIP transport protocols, not replacing them. AoIP protocols move audio around the facility over the network. MXL moves it around inside the software processes. The two are complementary.

It's also worth noting the governance model. MXL is Apache 2.0 licensed and maintained through collaborative engagement between broadcasters and technology suppliers - similar in spirit to how AES67 emerged as a common interoperability layer above competing proprietary protocols. That open approach is usually a good sign for long-term adoption.

The project is still relatively early - v1.0.0 landed in February 2026 - but the backing of organisations like the EBU and CBC suggests this is worth paying attention to. As more broadcast facilities move toward software-defined and cloud-based architectures, the question of how audio moves efficiently between processes is only going to become more pressing. MXL looks like a serious attempt to answer it.
