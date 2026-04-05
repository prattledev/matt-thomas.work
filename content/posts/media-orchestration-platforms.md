---
date: 2026-04-05T13:00:00Z
draft: false
categories:
  - Broadcast Technology
  - AoIP
tags:
  - ST 2110
  - NMOS
  - IP
  - Audio Engineering
  - Networking
title: "Media Orchestration Platforms in IP Broadcast Facilities"
description: "A media orchestration platform is the control layer that sits above an IP media network and manages routing, resources, and signal paths across it. Understanding what it does - and what it cannot do - is essential for anyone operating or designing ST 2110 infrastructure."
cover:
  image: videoipath_mop.png
  alt: "Media orchestration platform architecture diagram"
---

Building a spine-leaf ST 2110 network solves the transport problem. Signals can flow anywhere in the facility at the speed of light with predictable latency. What it does not solve is the control problem: how do operators actually route those signals, how does the system know what resources are available, and how does automation talk to the infrastructure in a consistent way regardless of which vendor made each device?

That is what a media orchestration platform does. It is the software layer that sits above the network and provides a unified means of controlling everything connected to it.

## What a Media Orchestration Platform Is

A media orchestration platform is a centralised control system for IP media infrastructure. It maintains a live model of all devices and resources on the network, provides path-finding across that network to connect sources to destinations, and exposes both a human-operated interface and an API for automation systems.

The key word is abstraction. Below the platform is a heterogeneous collection of hardware - NMOS-compliant IP devices, encoders, decoders, multiviewers, audio processing, and often legacy SDI routing infrastructure as well. Each piece of hardware has its own control interface and its own idea of how to represent its resources. The orchestration platform normalises all of that into a single model and presents operators and automation systems with a consistent view.

In a traditional SDI facility, the router control system performed a similar function for the routing layer. A media orchestration platform extends that concept to the full complexity of an IP media network, where a "route" is not a single crosspoint but a coordinated chain of operations: registering a flow, reserving network resources, configuring receivers, establishing multicast group membership, and verifying the path is carrying signal.

## Architecture

Most platforms share a common structural pattern, even when the implementation details differ between vendors.

**Resource registry** - The platform discovers and maintains a database of all devices, flows, and endpoints on the network. In an NMOS environment this is fed by IS-04, the discovery and registration API, which devices use to advertise their senders and receivers. The registry is the platform's model of what exists and what is available.

**Control engine** - The core of the platform. When a route is requested, the control engine calculates the path through the network, checks for conflicts with existing allocations, and sends the necessary control commands to each device involved. In an NMOS environment this uses IS-05, the connection management API, to activate flows on senders and configure receivers. For non-NMOS devices, proprietary drivers handle the translation.

**Southbound adapters** - The drivers that translate between the platform's internal model and the specific control interfaces of individual devices. A well-designed platform has adapter support for NMOS, SNMP, REST APIs from major hardware vendors, and legacy serial or GPI control for older infrastructure. The quality and breadth of adapter support varies significantly between platforms and is often where integration projects encounter problems.

**Northbound API** - The interface through which external systems talk to the platform. Production automation, scheduling systems, monitoring infrastructure, and custom tooling all interact here. A REST API is standard. Some platforms also expose NMOS IS-08 for audio channel mapping, allowing fine-grained control of channel assignments within flows.

**Operator interface** - A web-based GUI presenting a patchbay, a signal flow view, or both. Operators select sources and destinations, the platform resolves the path and executes the route. Status, alarms, and resource utilisation are surfaced here.

The platform itself typically runs on a server or virtual machine in the facility's IT infrastructure, often with a redundant standby instance. Some vendors offer containerised deployments for cloud or hybrid environments.

## What It Controls

**Signal routing** - The primary function. Connecting a source to one or more destinations, whether that is a single point-to-point route or a multicast flow with many receivers. The platform handles the difference between these transparently to the operator.

**Resource management** - Tracking encoder, decoder, and processing resources. In a facility where encode and decode capacity is limited, the platform arbitrates access to those resources, prevents double-booking, and surfaces availability to operators before they attempt a route.

**Hybrid SDI/IP routing** - Most facilities running ST 2110 still have SDI infrastructure. A mature orchestration platform controls both, treating SDI router crosspoints and IP connection management as part of the same routing model. From an operator's perspective, a signal is a signal regardless of whether it is moving over SDI or IP.

**Audio channel mapping** - Platforms supporting NMOS IS-08 can control which audio channels within a multi-channel flow are assigned to which physical outputs. This matters for facilities where the channel order within an AES67 or ST 2110-30 stream needs to be rearranged before it reaches a console or monitor.

**Path reservation and scheduling** - In more sophisticated deployments, the platform supports pre-booking signal paths for scheduled events. Automation triggers the route at the right time without operator intervention.

**Monitoring integration** - While monitoring is usually handled by a separate system, orchestration platforms typically surface device status, flow health, and alarm states. Some integrate directly with monitoring platforms to correlate signal loss events with the current routing state.

## Nevion VideoIPath

Nevion VideoIPath is one of the most widely deployed media orchestration platforms in professional broadcast. It is built around NMOS and designed specifically for the control challenges of ST 2110 environments, though it supports hybrid SDI/IP configurations.

VideoIPath uses NMOS IS-04 for discovery and IS-05 for connection management natively. Its path-finding engine handles the complexity of routing across a spine-leaf fabric, including managing multicast group membership and coordinating endpoints from multiple vendors in a single route. IS-08 support enables audio channel mapping as part of the routing workflow.

The platform exposes a REST API for integration with production automation and third-party systems. A number of major broadcast facilities and outside broadcast operators use VideoIPath as the control layer for live production infrastructure, and it is regularly specified on large ST 2110 facility builds.

Other platforms operating in the same space include Grass Valley GV Orbit, Imagine Communications Selenio Network Processor and its associated control layer, and Ross Video's openGear-based orchestration tooling. Each has different strengths in terms of hardware ecosystem support, so the choice often follows the hardware already in the facility.

## Advantages

**Vendor-agnostic control** - A well-integrated orchestration platform means operators do not need to know or care which vendor made each device. The routing model is consistent regardless of whether the signal is traversing Arista switches, Sony cameras, Lawo audio processors, and Evertz multiviewers, or any other combination.

**Operator simplicity** - The complexity of IP routing - flow activation, multicast management, receiver configuration - is handled by the platform. Operators see a patchbay. The network engineering required to make a route happen is abstracted away.

**Conflict management** - The platform knows the current state of every resource. It can prevent a route that would exceed decoder capacity, flag a path that is already in use, or alert an operator before they disrupt an on-air signal. This is categorically not possible when operators are making routes directly through individual device interfaces.

**Automation integration** - A northbound API turns the platform into a controllable resource for production automation, rundown systems, and custom tooling. Routes can be executed programmatically at scale without operator intervention, which is what makes large IP facilities operationally viable.

**Audit trail** - Every route change is logged. For live production environments where things go wrong at the worst possible times, knowing exactly what changed and when is valuable.

## Disadvantages

**Single point of failure risk** - The platform is a critical dependency for the entire facility. If the orchestration platform fails, making routes typically requires falling back to direct device control interfaces, which operators may not be trained on and which do not provide the same level of conflict management. Redundant platform instances are standard practice in serious deployments but add cost and complexity.

**Adapter quality varies** - The southbound adapter layer is where integrations succeed or fail. Official support from the platform vendor for a given device does not guarantee that all of that device's capabilities are exposed. Newer features, unusual configurations, and devices from smaller vendors are common pain points. Budget significant testing time for any integration project.

**Operational complexity** - The platform requires commissioning, configuration, and ongoing maintenance by people who understand both broadcast and IP. Misconfiguration at the platform level can produce routing failures that are harder to diagnose than failures in simpler systems, because the abstraction layer obscures where in the chain the problem actually is.

**Lag behind hardware** - Device firmware updates sometimes introduce new capabilities or change API behaviour in ways that break existing platform integrations. Keeping platform adapters current with hardware releases is ongoing work and depends on the vendor's release cycle.

**Cost** - Licensing a production-grade media orchestration platform is a significant line item. For smaller facilities, the cost and operational overhead may not be justified by the scale of the routing problem they are solving.

## From an Audio Engineering Perspective

For audio engineers, the orchestration platform is the system through which AES67 and ST 2110-30 sources and receivers are connected. Understanding it matters because audio routing problems often manifest at the platform level - a route that fails, an IS-05 connection that activates but carries no audio, a channel mapping that is wrong - and diagnosing them requires knowing whether the problem is in the platform's model, the adapter, the device itself, or the network.

The IS-08 layer is particularly relevant. In facilities where multi-channel audio flows carry more channels than any individual destination needs, IS-08 is what determines which channels end up where. If the audio arriving at a console or monitor is on the wrong channels, or channels are missing, that is where to look.

The platform is also the right place to start when investigating signal flow for unfamiliar productions. A well-maintained orchestration system is a live map of what is connected to what. That information is significantly more useful than tracing cable and hoping the label is current.
