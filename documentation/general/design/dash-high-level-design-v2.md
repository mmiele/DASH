---
version: 0.2
description: DASH architecture draft
last update: 03/10/2022
---

# DASH architecture

- [Introduction](#introduction)
  - [Objectives](#objectives)
- [Requirements](#requirements)
- [Implementation scenarios](#implementation-scenarios)
- [Use scenarios](#use-scenarios)
- [Ecosystem](#ecosystem)
- [Logical architecture](#logical-architecture)
    - [SDN controller](#sdn-controller)
      - [SDN and DPU High-Availability (HA)](#sdn-and-dpu-high-availability-ha)
  - [DASH container](#dash-container)
    - [Multiple DPUs device](#multiple-dpus-device)
  - [SONiC Containers](#sonic-containers)
  - [Switch State Service (SWSS)](#switch-state-service-swss)
  - [Switch Abstraction Interface (SAI) DASH](#switch-abstraction-interface-sai-dash)
  - [ASIC Drivers](#asic-drivers)
  - [DASH capable ASICs](#dash-capable-asics)
- [SONiC integration](#sonic-integration)
- [Physical architecture](#physical-architecture)
  - [Deployment](#deployment)
- [API](#api)
  - [SAI API](#sai-api)
    - [Overlay](#overlay)
    - [Underlay](#underlay)
  - [DASH pipeline API](#dash-pipeline-api)
    - [Behavioral model](#behavioral-model)
  - [Test](#test)
- [Appendix](#appendix)
  - [Single DPU on NIC](#single-dpu-on-nic)
  - [Appliance](#appliance)
  - [Smart switch](#smart-switch)
  - [A day in the life of a DASH packet](#a-day-in-the-life-of-a-dash-packet)
  - [A day in the life of a DASH SDN controller](#a-day-in-the-life-of-a-dash-sdn-controller)
  - [A day in the life of a DASH container](#a-day-in-the-life-of-a-dash-container)
- [References](#references)

## Introduction

![dash-words-cloud](../images/general/dash-words-cloud.png)

### Objectives
The overall objective is to **optimize network SMART Programmable Technologies performance**, and **leverage commodity hardware technology** to achieve **10x or even 100x stateful connection performance**.

- With the help of network hardware technology suppliers, create an open forum that capitalizes on the use of **programmable networking hardware** including SmartNICs, SmartToRs, SmartAppliances. 
- Optimize **stateful L4** performance and connection scale by 10x or even 100x when compared to implementations that make extensive use of a generic software stack approach that compromises performance for flexibility.  This flexibility was needed early on, whereas the cloud is maturing and is ready for a further optimized approach.  As host networking in the cloud is performed at L4, the resulting performance improvements should be truly significant.
- Microsoft Azure will integrate and deploy DASH solutions to ensure that scale, monitoring, reliability, availability and constant innovation are proven and hardened. Other enterprise and cloud providers may deploy DASH as well, and we hope to hear similar feedback and contributions as we move forward.  It should be noted that innovations for **in-service software upgrades** (ISSU) and **high availability** (HA) are key tenets of the DASH charter. 
  
## Requirements

From SDN packets transform document.

## Implementation scenarios 

From SDN packets transform document. This is what the technology providers are working on. 

## Use scenarios 

Using Azure portal to provision DASH capable devices.
We just capture the gist of these scenarios and then we link to the related documentation. 

## Ecosystem
 
Most of the stuff we talked about at the coffee shop:

1. Some of your knowledge about the system.
1. Some of the info from the doc you *sanitized*. 

## Logical architecture  

 DASH builds upon the traditional SONiC architecture, which is documented in the SONiC Wiki under [Sonic System Architecture](https://github.com/Azure/SONiC/wiki/Architecture#sonic-system-architecture). The following descriptions assume familiarity with the SONiC architecture and will describe DASH as incremental changes relative to traditional SONiC. Notice that DASH adds a new **SDN control plane** via **gNMI** in the **DASH container**. The following figure depicts DASH software stack. 

![dash-software-stack](images/architecture/dash-software-stack.svg)

<figcaption><i>Figure 1 - DASH software stack</i></figcaption><br/>

#### SDN controller

The SDN controller is **primarily responsible for controlling the DASH overlay services**, while the traditional SONiC application containers are used to manage the underlay (L3 routing) and hardware platform. Both the DASH container and the traditional SONiC application containers sit atop the Switch State services (SWSS) layer, and manipulate the Redis application-layer DBs; these in turn are translated into SAI dataplane obects via the normal SONiC orchestration daemons inside SWSS.

The SDN controller controls the overlay built on top of the physical layer of the infrastructure.  From the point of view of the SDN control plane, when a customer creates an operation, for example a VNET creation, from the cloud portal, the controller allocates the resources, placement management, capacity, etc. via the  **NorthBound interface APIs**.

##### SDN and DPU High-Availability (HA)

For **High Availability** (HA), the SDN controller selects the pair of cards and configures them identically.  The only requirement on the card from the HA perspective is for the cards to setup a channel between themselves for flow synchronization.  The synchronization mechanism is left for technology suppliers to define and implement. For more information, see [High Availability and Scale]() document.   

### DASH container
 
The SDN controller communicates with a DASH device through a **[gNMI](https://github.com/Azure/DASH/wiki/Glossary#gnmi) endpoint** served by a new DASH SDN agent **running inside a new SONiC DASH container**.  

In summary:
- The DASH container translates SDN configuration modeled in gNMI into **SONiC DB** objects. The gNMI schema is closely related to the DASH DB schema so in effect, the gNMI server is a a thin RPC shim layer to the DB.
- The **SONiC orchagent** inside the Switch State Service (SWSS) Container will be enhanced to transform and translate these objects into **SAI_DB objects**, including the new **DASH-specific SAI objects**.  
- An **enhanced syncd** will then configure the dataplane using the **technology supplier-specific SAI library**.

A **gNMI schema** will manage the following DASH services: 
- Elastic Network Interface (ENI)
- Access Control Lists (ACLs) 
- Routing and mappings
- Encapsulations 
- Other  

See also [TBD gnmi schema]().

#### Multiple DPUs device

In the case of a multiple DPUs device the following applies:

- Each DPU provides a gNMI endpoint for SDN controller through a unique IP address. 
- An appliance or smart switch containing multiple DPUs therefore contains multiple gNMI endpoints for SDN controller, and the controller treats each DPU as a separate entity. 
- To conserve IPv4 addresses, such an appliance or switch _might_ contain a proxy (NAT) function to map a single IP address:port combination into multiple DPU endpoints with unique IPv4 addresses.  
- No complex logic will run on the switches (switches do not have a top-level view of other/neighboring switches in the infrastructure). 

### SONiC Containers

 In the figure above, the "SONiC Containers" box comprises the normal collection of optional/customizable application daemons and northbound interfaces, which provide BGP, LLDP, SNMP, etc, etc. These are described thoroughly in the [Sonic System Architecture](https://github.com/Azure/SONiC/wiki/Architecture#sonic-system-architecture) wiki and reproduced in diagram form under the [Detailed Architectures](#detailed-architectures) section of this document.
### Switch State Service (SWSS)
The SWSS container comprises many daemons which operate on conceptual SONIC config objects across several databases. DASH affects the following daemons, as follows:
* `orchagent`, which translates `XX_DB` objects (application and state DBs - **TODO** - identify) into `ASIC_DB` objects, must be enhanced to manage new DASH overlay objects, such as ACL1,2,3 rules, ENI mappings, etc. The `orchagent` has to manage both representations of SONiC objects (`XX_DB` and `ASIC_DB`) and translates between them bidirectionally as required.
* `syncd`, which translates `ASIC_DB` conceptual objects into technology supplier SAI library API calls, must likewise be enhanced to handle new DASH SAI objects.

### Switch Abstraction Interface (SAI) DASH

The Switch Abstraction Interface (SAI) is a common API that is supported by many switch ASIC technology suppliers. SONiC uses SAI to program the ASIC. This enables SONiC to work across multiple ASIC platforms naturally. DASH uses a combination of traditional SAI headers and new DASH pipeline-specific headers. Technology suppliers must implement this interface for their DASH devices. This is the primary integration point of DASH devices and the SONiC stack. It will be rigorously tested for performance and conformance. See [DASH Testing documentation](https://github.com/Azure/DASH/tree/main/test).

SAI "schema" are represented as fixed c-language header files and derived metadata header files. The underlay and overlay schema have different origins:
* Traditional SAI headers are defined in the [OCP SAI project repo](https://github.com/opencomputeproject/SAI/tree/master/inc).These are hand-generated and maintained. DASH uses a subset of these to manage underlay functions, e.g. device management, Layer 3 routing and so forth.
* DASH SAI "overlay" objects are derived from a [P4 Behavioral Model](https://github.com/Azure/DASH/tree/main/sirius-pipeline). A script reads the P4 model and generates SAI header files.

DASH uses an **enhanced syncd** to configure the dataplane using the technology supplier-specific SAI library.

### ASIC Drivers
The term "ASIC Drivers" is borrowed from traditional SONiC and SAI, where a datacenter switch was implemented almost entirely inside an ASIC (or multiple ASICs). These devices are programmed using a technology supplier Software Development Kit (SDK) which includes device drivers, kernel modules, etc.

A contemporary DASH "SmartNIC" may consist of many complex hardware components including multi-core System On A Chip (SoC) ASICs, and the associated software. For simplicity, the software for such systems which interfaces to the SAI layer is collectively called the "ASIC driver." More importantly, the technology supplier SAI library will hide all details and present a uniform interface.

### DASH capable ASICs
These comprise the main dataplane engines and are the core of what are variously called SmartNICs, DPUs, IPUs, NPUS, etc. The actual cores may be ASICs, SoCs, FPGAs, or some other high-density, performant hardware.

## SONiC integration

This is what Prince is doing. For us it is important to capture how SONiC is modified to accomodate DASH. We'll to the related SONiC docs when needed. I put all the varaitions in the appendix so we do not clutter the flow of essential info. 

![dash-high-level-design](images/architecture/dash-high-level-design.svg)

<figcaption><i>Figure 2 - SONiC integration</i></figcaption>

## Physical architecture  

### Deployment


## API

We describe in general terms the intent and where it *fits* in the *big picture*.
### SAI API

We just capture the gist of it and then we link to the related SAI API. 

#### Overlay 


#### Underlay

### DASH pipeline API

Previousley named *Sirius pipeline API*.

We just capture the gist of it and then we link to the related Sirius pipeline API. 

#### Behavioral model



### Test

We just capture the gist of it and then we link to the related test area. 

## Appendix

### Single DPU on NIC


### Appliance


### Smart switch 

### A day in the life of a DASH packet

### A day in the life of a DASH SDN controller

### A day in the life of a DASH container


## References

