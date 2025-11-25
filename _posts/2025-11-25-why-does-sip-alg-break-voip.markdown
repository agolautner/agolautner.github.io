---
layout: post
title:  "Conceptual: Why does SIP ALG break VoIP services?"
date:   2025-11-25
categories: portfolio
---
One typical issue any professional working with IP telephony is almost guaranteed to encounter is that networks with a certain setting enabled produce strange behavior during VoIP calls. **SIP Application Level Gateway (ALG)**, or in some contexts, **SIP Inspection**, has gained notoriety for causing unexpectedly dropped calls, loss of media, and calls that cannot be answered, among other things.

## What is SIP ALG trying to solve?

Ironically enough, SIP ALG was originally designed in an attempt to solve a long-standing issue in VoIP telephony, encountered when operating on networks utilizing the **Network Address Translation (NAT)** protocol.

NAT, which is an indispensable tool on public-facing networks, allows multiple devices on the same network to share a single public IP address through port-based translation. Traffic from outside of the network can then be routed to individual devices or services based on the NAT device's translation rules.
This achieves two goals: it offers extra security and privacy by masking the private IP address of devices on the network (although it can be argued that this is not as much an actually intended goal as it is a positive side-effect), and it reduces the number of IPv4 addresses required.

However, this behavior does not play well with the **Session Initiation Protocol (SIP)**. SIP is a text-based protocol, and is used to establish, manage and tear down communication sessions between parties.
In layman's terms, it lets devices introduce themselves to each other (often trough a "friend in common", a SIP proxy) and  let the other know that they would like to start communicating. Additionally, it lets them negotiate how they will communicate - what codecs to use, which ports to send audio to, etc., by carrying **Session Description Protocol (SDP)** payloads. Think of this as people agreeing on what language they should speak to each other by exchanging a list of languages they each speak, and how well.

NAT, which hides private IP addresses from the public internet, has long been known to mess with SIP communication. Important pieces of information, such as IP addresses and port numbers get quite literally lost in translation, and SIP messages end up containing the wrong "return address".
When a device "introduces itself" via SIP, the SIP/SDP payload often includes its private IP address. When the message goes trough the NAT translation step, the NAT device alters the IP packet header and replaces the private address with a public one. It does not, however, alter the actual payload, leaving the private IP unaltered.
When the SIP message reaches its destination, the party on the other side attempts to communicate with the private address it receives - which it obviously cannot do. 

Thus, we end up with calls that don't seem to go through, media that seems to get lost on the way, and similar annoyances.

## What does SIP ALG do?

So how does SIP ALG try to remedy this issue? We already know that the NAT device modifies the IP packet's headers, masking the private IP address, but it leaves the payload unaltered, leading to the issues discussed above.

This is where SIP ALG comes in. ALG inspects the contents of each packet on the application layer - meaning it actually looks into the SIP message - and replaces the private address with the public one. 

*At least in theory.*

In practice, what often happens is it replaces occurrences of the private IP in some SIP headers (such as *Via* and *Contact*), while often completely ignoring or mishandling the SDP body, where the actual media channels are established. So, while the SIP message may actually arrive to the destination, it will instruct the receiver to send its media to a private IP address that it cannot communicate with - essentially telling it to throw them out the window.

This can explain the typical occurrence of a call being active, but no audio going through in one (or both) directions: the two parties receive each other's SIP messages and agree to start a session, but they are sending the actual media packets into the void.

## The good news

Fortunately, the solution for the issues caused by SIP ALG is quite simple in most cases: just turn the setting off.

Truth is, SIP ALG is a stubborn remainder of technology that is no longer very relevant. Modern VoIP solutions, such as Teams and FreePBX handle NAT traversal themselves, using more modern protocols, such as **Session Traversal Utilities for NAT **, **Traversal Using Relays around NAT (TURN)** and **Interactive Connectivity Establishment (ICE)**.
