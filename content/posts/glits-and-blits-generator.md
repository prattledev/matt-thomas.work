---
date: 2026-04-26T12:00:00Z
draft: false
categories:
  - Broadcast Technology
  - Tools
tags:
  - Audio Engineering
  - Broadcast
  - Tools
  - Test Tones
title: "GLITS & BLITS Generator"
description: "A browser-based tool for generating broadcast-standard GLITS and BLITS test tone sequences as 24-bit WAV files - without needing a DAW or dedicated hardware."
cover:
  image: glitsandblits.png
  alt: "GLITS & BLITS Generator"
---

GLITS and BLITS are two broadcast-standard test tone sequences that most audio engineers have encountered but few have had a convenient way to generate from scratch.

**GLITS** - Group Line-up and Identification Tone Sequence - is the BBC/EBU stereo alignment standard. It runs as a 4-second cycle of 1 kHz tone at -18 dBFS on both channels, with timed interruptions on each channel in turn to identify left and right. It is the tone you hear at the top of a contribution feed or at the start of a tape, and it tells you the line is up, the levels are correct, and the channels are the right way round.

**BLITS** - Bass Management and Level Identification Tone Sequence - is the EBU Tech 3304 equivalent for 5.1 surround. It runs sequential 600 ms identification bursts, one per speaker position, using specific frequencies assigned to each channel, across a full 13.4-second cycle. It is used to verify that a 5.1 path is correctly routed end to end - that the left surround is arriving at the left surround, that the LFE is reaching the subwoofer, and that nothing has been swapped or dropped in the chain.

Both are well-defined standards. Generating them correctly - at the right level, with the right timing, with proper WAVE_FORMAT_EXTENSIBLE headers at 24-bit/48 kHz - has historically required either a DAW session, dedicated signal generator hardware, or a file sourced from somewhere else and hoped to be correct.

So I built a tool to generate them directly in the browser.

**[GLITS & BLITS Generator](https://github.com/prattledev/glits-and-blits-gen)** is a simple web application that produces compliant GLITS and BLITS sequences as 24-bit PCM WAV files at 48 kHz. Select the sequence type, set the number of repetitions, and download. All other parameters follow the EBU specifications and are not configurable - the point is to generate a correct, standard file, not to build a general-purpose tone generator.

The source is on GitHub and the project is MIT licensed.
