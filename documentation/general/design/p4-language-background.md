[<< Back to parent directory](../README.md) ]

[<< Back to DASH top-level Documents](../../README.md#contents) ]


# P4 language

This article is intended to give you an overiew of the **Programming
Protocol-independent Packet Processors** (P4) language. For more in depth information,
please refer to the official documentation [P4 Open-Source
Programming Language](https://p4.org/).

> [!NOTE] The DASH project does not provide and/or mandate any P4 implementation.
> Instead, it uses P4 to **define the dataplane behavioral model** as the "gold standard".
> The technology providers use this model to implement their dataplane logic.
> Notice that this implementation does not have to be in P4.

## Overview

P4 is a domain-specific language for network devices, specifying how data plane
devices (switches, NICs, routers, filters, etc.) process packets. It is designed
to be implementable on a large variety of targets including programmable
**network interface cards** (NIC), **FPGAs**, **software switches**, and
**hardware ASICs**. As such, the language is restricted to constructs that can
be efficiently implemented on all of these platforms.

Before we go any further, let's have a couple of definitions.

- **Control plane**. It comprises a class of algorithms and the corresponding input and output data that are concerned with the **provisioning** and **configuration** of the **data plane**. 
- **Data plane**. It comprises a class of algorithms that describe **transformations** on **packet by packet processing systems**. 

P4 is concerned about the data plane that is the **transformations** on **packet by packet processing systems**. 
