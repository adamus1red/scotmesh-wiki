---
title: TX Delay and Flood Propagation on ScotMesh
description: Explains ho TX delay can help avoid  'round the houses' paths
published: true
date: 2026-08-30T18:04:01.111Z
tags: 
editor: markdown
dateCreated: 2026-08-30T18:01:48.629Z
---

# TX Delay and Flood Propagation on ScotMesh

> **Key point:** TX Delay scales a randomized waiting window before a repeater forwards flood traffic. A lower value creates a smaller window and can statistically bias that repeater toward earlier forwarding. It is not a routing table, does not prescribe a route, and does not guarantee which repeater transmits first.

## Purpose and scope

This page explains how ScotMesh repeater operators can use TX Delay as a coordinated network-planning setting. It combines the current MeshCore implementation and command reference with a practical decision flow derived from Sydney Mesh guidance and published by the Island of Ireland LoRa Project.

The recommendations are useful defaults, not automatic proof that a site should receive a particular setting. RF coverage, neighbour relationships, network density, firmware behaviour and observed traffic should all be reviewed before changing a deployed repeater.

## What TX Delay actually does

For flood traffic, a MeshCore repeater estimates the packet airtime, multiplies it by the configured TX Delay factor, and chooses a random retransmission delay from a window based on that result. In the current repeater source, the effective calculation is:

```text
t = estimated packet airtime x txdelay
random delay = 0 to (5 x t), inclusive in the implementation
```

The random waiting window is primarily a collision-avoidance mechanism. When several nearby repeaters hear the same flood packet, spreading their forwarding times reduces the chance that they all transmit simultaneously.

A larger factor widens the random window and can reduce collision risk, but adds potential latency. A lower factor narrows the window and makes earlier forwarding statistically more likely, but never certain.

### Worked example

For illustration, if a packet has an estimated airtime of 500 ms, the approximate random forwarding windows are:

| TX Delay | Approximate random forwarding window |
|---:|---:|
| 0.25 | 0-625 ms |
| 0.3 | 0-750 ms |
| 0.6 | 0-1,500 ms |
| 1.0 | 0-2,500 ms |
| 1.4 | 0-3,500 ms |
| 2.0 | 0-5,000 ms |

*Illustrative values calculated from the current source formula; actual airtime depends on the packet and radio parameters.*

> **Important limitation:** Do not describe TX Delay as a mechanism that necessarily cancels a scheduled retransmission when another repeater is heard. The current TX Delay setting controls the random retransmission window. A separate proposal to add configurable suppression after hearing other repeats shows that this behaviour should not be attributed to TX Delay itself.

## Timing influence, not route selection

MeshCore flood propagation is not conventional shortest-path routing. Multiple repeaters may receive and forward a flood packet, and the observed path is shaped by RF reception, topology, packet timing, hop limits and the behaviour of participating nodes.

Because lower-delay repeaters draw from a smaller random window, a coordinated hierarchy can bias strategically useful infrastructure toward earlier forwarding opportunities. This is a probability effect across repeated events, not a priority flag.

It does not guarantee that a hilltop repeater wins any particular forwarding race, that another repeater will remain silent, or that a packet follows the geographically shortest path.

## The Sydney-derived TX Delay decision flow

The Island of Ireland LoRa Project publishes a sequential decision flow based on Sydney Mesh advice and adapted for its own network. For ScotMesh, the same flow is best treated as a starting framework: ask the questions in order and stop at the first **Yes** answer.

> **Why order matters:** A newly installed mountaintop repeater still matches “Just placed - under 3 days?” before it reaches the mountaintop question. The conservative `1.4` setting therefore comes first while its real RF role is observed.

| Step | Question - stop at the first **Yes** | Suggested `txdelay` |
|---:|---|---:|
| 1 | First repeater setup? | **1.4** |
| 2 | Temporary or short-term placement? | **1.4** |
| 3 | Just placed - under 3 days? | **1.4** |
| 4 | Few neighbours - 3 or fewer? | **1.4** |
| 5 | Attic, window or below roofline? | **1.0** |
| 6 | Primarily connects its owner to the mesh? | **1.0** |
| 7 | Above-roofline mast or top of apartment block? | **0.6** |
| 8 | Hill overlooking a neighbourhood? | **0.6** |
| 9 | Mountaintop with views for miles? | **0.3** |
| 10 | Directional antenna or specialist link? | **0.25** |
| 11 | Mobile or vehicle repeater? | **2.0** |
| Else | None of the above | **1.4** |

*These values reproduce the published decision flow. They are community planning guidance, not MeshCore firmware defaults and not a guarantee of path behaviour.*

### How to interpret the values

- **2.0 - mobile or vehicle:** a deliberately broad window for a changing, non-fixed RF role.
- **1.4 - new, temporary, lightly connected or unclassified:** the conservative starting point used by the published flow.
- **1.0 - domestic or local-access site:** typically below roofline or mainly connecting one operator.
- **0.6 - useful elevated local infrastructure:** above-roofline mast, apartment-block roof or neighbourhood hill.
- **0.3 - major wide-area mountaintop infrastructure:** after the site's role has been demonstrated.
- **0.25 - directional or specialist link:** reviewed as part of the wider topology.

A lower number is not an upgrade. It represents a narrower timing window and should be justified by the repeater's stable, observed role.

## Geographically surprising paths

Observed flood paths can appear indirect on a map even when every individual RF hop is valid. The image below is a real ScotMesh example in western Scotland.

![scotmesh-tx-delay-route.jpg](/scotmesh-tx-delay-route.jpg)

*Figure 1. Example of an observed ScotMesh path in western Scotland: MUIRSHIEL-TT1 -> ARDROSSAN-RPT -> AYR-RPT -> GALLOWAY-TT1. The path is geographically indirect, but the image alone does not show that any RF link or repeater is faulty.*

This example demonstrates why a flood path should not be interpreted as a conventional shortest route. Coordinated TX Delay values may influence the relative timing of repeaters participating in the flood and may make some forwarding patterns more or less likely over time. They do not prescribe this path, prevent it, or guarantee a different one.

> **Read the evidence carefully:** Use path traces as a prompt for investigation, not as proof that one TX Delay value caused the route. Compare repeated observations, neighbour data, RF coverage and traffic conditions before changing settings.

## Practical guidance for ScotMesh

### For a new or newly moved repeater

- Start at `1.4` and leave the repeater in its intended permanent position.
- Keep it at `1.4` for at least the first three days, consistent with the published decision flow.
- Record its neighbours, reliable coverage and repeated path observations; avoid judging it from a single trace.
- Classify the site by what it demonstrably contributes to the network, not only by height or predicted coverage.
- Coordinate any reduction with nearby ScotMesh operators, particularly where coverage overlaps.

### For an established repeater

- Review whether the current value matches the site's actual role and stability.
- Change one variable at a time, note the date and reason, then compare behaviour over a meaningful observation period.
- Do not lower values simply to make a node “win”. Too many narrow windows can increase simultaneous competition and undermine collision avoidance.
- If a path is surprising, check RF reachability, neighbour counts, firmware versions, scope and hop settings alongside TX Delay.

## CLI examples

Read the current flood TX Delay value:

```text
get txdelay
```

Set the conservative starting value:

```text
set txdelay 1.4
```

Example for an established elevated local site after review:

```text
set txdelay 0.6
get txdelay
```

The current MeshCore CLI documents TX Delay as a factor from `0` to `2`. The commands above concern flood traffic. Direct, non-flood routed traffic uses the separate `direct.txdelay` setting; do not change it merely to mirror this guide.

## Review checklist

- Is the repeater permanently installed and observed for at least three days?
- Does it have more than three useful neighbours?
- Is the proposed classification supported by real RF coverage?
- Have repeated traces been reviewed rather than a single path?
- Have neighbouring operators considered the change as a coordinated system?
- Is the change recorded so its effect can be compared later?

## Summary

> **In one line:** TX Delay -> randomized flood retransmission window -> statistical timing bias -> possible influence on flood propagation; never a guaranteed route.

Start conservatively, observe the site, and coordinate lower settings where the repeater's stable network role justifies a narrower random window. Treat unexpected routes as evidence to investigate, not behaviour that TX Delay alone can promise to fix.

## Credits and sources

The decision flow and community guidance are based on the [Island of Ireland LoRa Project's Repeater Setup and TX Delay guide](https://loraproject.ie/repeater/#tx-delay). That page credits Sydney Mesh advice as the basis for its recommendations, adapted for Ireland.

- [MeshCore CLI commands - TX Delay](https://github.com/meshcore-dev/MeshCore/blob/main/docs/cli_commands.md#view-or-change-the-retransmit-delay-factor-for-flood-traffic)
- [MeshCore simple repeater source - `MyMesh.cpp`](https://github.com/meshcore-dev/MeshCore/blob/main/examples/simple_repeater/MyMesh.cpp)
- [MeshCore issue #2442 - proposal for repeat suppression](https://github.com/meshcore-dev/MeshCore/issues/2442)

*Source and firmware behaviour checked 30 August 2026. Review this page when MeshCore forwarding behaviour or CLI semantics change.*
 content here