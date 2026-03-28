---
date: '2026-03-28T12:00:00+00:00'
draft: false
categories:
- Broadcast Technology
- AoIP
tags:
- MADI
- AoIP
- Audio Engineering
- Opinion
title: 'Why MADI Still Matters'
description: "AoIP is transforming broadcast audio, but MADI remains a practical, reliable choice for many productions. Here's why it's not going anywhere yet."
---

The audio world has its own version of the IP debate, and if you've spent any time around broadcast engineers in the last few years, you'll have heard it. MADI is old technology. AoIP is the future. Dante, Ravenna, AES67 - that's where everything is heading, and anyone still specifying MADI is stuck in the past.

I've heard this argument a lot. I've also spent years working with both technologies at major live events, and my view is considerably more nuanced.

For context: I build tools for Ravenna. I think AoIP is genuinely transformative. And I still specify MADI on a regular basis.

## Why MADI keeps showing up

MADI does something that still feels almost remarkable when you think about it - it carries 64 channels of audio down a single coax or fibre cable with no configuration whatsoever. No network, no IP addresses, no clocking negotiation beyond a basic word clock or self-clocking setup. You plug it in and it works.

On an OB truck or in a temporary broadcast centre, that simplicity is worth an enormous amount. When you're building a system fast, with a mixed crew, under pressure, the last thing you want is a technology that requires careful network architecture to function correctly. MADI has no such requirements. Every broadcast engineer knows it, most equipment still supports it, and the failure modes are completely understood.

There's also the latency argument. MADI is low and consistent. For comms and IFB systems in particular, where even small timing variations are noticeable, that predictability matters.

## Where AoIP genuinely wins

I'm not making a case against AoIP - I wouldn't have built RavennaCalc if I didn't believe in it. The flexibility that Ravenna and AES67 offer is real and significant.

The ability to route audio across a standard IP network, reconfigure signal paths in software, and share infrastructure with other data traffic is a genuine step forward. For large-scale events where you're moving hundreds of channels between venues, or for remote production workflows where audio needs to travel over WAN connections, AoIP opens up options that simply don't exist with MADI.

Channel density is another factor. MADI tops out at 64 channels per link. An IP network scales as far as your bandwidth does.

## The honest answer

MADI and AoIP aren't really competing for the same jobs anymore - at least not in my experience. MADI is where you want reliability, simplicity, and near-zero configuration overhead. AoIP is where you need scale, flexibility, or integration with IP infrastructure.

Most of the systems I work on use both. MADI between audio consoles and intercom systems, Ravenna for distribution and remote feeds. That's not a compromise - it's just using the right tool for each part of the job.

What I'd push back on is the idea that MADI is somehow beneath serious consideration. It's a mature, stable, well-supported technology that solves a specific set of problems very well. In broadcast, that tends to have a long shelf life.
