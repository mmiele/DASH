---
Note: A dialog about DASH key archtectural elments
Last update: 06/09/2022
---

[<< Back to parent directory](../README.md) ]

[<< Back to DASH top-level Documents](../../README.md#contents) ]

# DASH key architectural elements

- [SONiC integration](#sonic-integration)
- [Packet flow](#packet-flow)
- [Processing pipeline](#processing-pipeline)
- [References](#references)

This article describes some of DASH key architecturale elements.

## SONiC integration

DASH relies upon the [SONiC system
architecture](https://github.com/Azure/SONiC/wiki/Architecture) as shown in the figure below.
For more information and details about the integration, see [SONiC DASH
HLD](https://github.com/Azure/DASH/blob/main/documentation/general/design/dash-sonic-hld.md). 

![dash-high-level-diagram](./images/hld/dash-high-level-design.svg)

<figcaption><i>Figure 1 - DASH architectural modifications to SONiC</i></figcaption><br/>

The previous figure shows the SONiC architectural modifications for DASH summirized below. 

1. **SDN controller**. The SDN controller is primarily **responsible for
   controlling the DASH overlay services**, while the traditional SONiC
   application containers are used to manage the underlay (L3 routing) and
   hardware platform. 
      1. A **DASH API** is exposed as **gNMI interface** as part of the *gNMI
   container*, see next point. 
      1. **gNMI client** configures SONiC via `gRPC get/set` calls.
   
   The SDN controller controls the overlay built on top of the physical layer
   (underlay) of the infrastructure. From the point of view of the SDN control
   plane, when a customer creates an operation from the cloud portal (for
   example a VNET creation), the controller allocates the resources, placement
   management, capacity, etc. via the NorthBound interface APIs.

2. **gNMI container**. The SDN controller communicates with a DASH device
   through a **gNMI endpoint** served by a new DASH SDN agent running inside a
   new SONiC **gNMI container**.  
   1. **gNMI server**. The gNMI schema is closely related to the DASH DB schema
   so in effect, the *gNMI server* is a **thin RPC shim layer** to the DB.
   1. **Config Backend**. Translates SDN configuration into **confdb** OR
      **APPDB** objects.

3. **Switch State Service (SWSS) container** **DASH underlay** has a small
   initialization and supports a defined set of **SAI APIs**.
   1. **DASH orchestration agent (dashorch)**. **DASH Overlay** in the SWSS
    container that subscribes to the DB objects programmed by the **gNMI
    agent**. It transforms and translate these objects into ASIC_DB objects,
    including the new DASH specific SAI objects.
   2. **DASH orchestration agent (orchagent)**. It writes the state of each
      tables to STATEDB used by the applications to fetch the programmed status
      of DASH configured objects.
4. **sync-d container**. Provides **sai api DASH** that uses the orchestration
  agent ASICDB to allow the technology providers to program the DPU via their
  SAI implementation. This is a DASH enhanced sync-d that configures the
  dataplane using the technology supplier-specific SAI library.

  Both the DASH container and the traditional SONiC application containers sit
  on top of the Switch State services (SWSS) layer, and manipulate the Redis
  application-layer DBs; these in turn are translated into SAI dataplane obects
  via the normal SONiC orchestration daemons inside SWSS.



DASH introduces the following modifications:

1. A *new docker container* in the user space named **gNMI container** (aka SDN
   container) to create the functional component for DASH. Microsoft will
   deliver the **gNMI container** as code to SONiC to allow any SONiC switch to
   talk with and integrate DPU technology. The *DASH container* software
   integrates with the SONiC system containers seemlessly. Microsoft will ensure
   a high quality integration with the switch. 

2. In the **sync-d container**, the **sai api DASH** (as opposed to *sai api* in
   the original SONiC architecture).  

In summary, the functionality of the new **gNMI container** in the user space is
to receive content from the Software Defined Networking (SDN) controller to
control setup for the overlay configurations. DASH receives the objects,
translates them with a **gNMI agent**, provides them to the *SONiC OrchAgent*
for further translation onto the dataplane via the **SAI database**. The
*DPU/IPU/SmartNic* device will run a separate instance of SONiC DASH on the
device.  



## Packet flow


## Processing pipeline




## References
