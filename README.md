# TF6310-TCP-IP sACN

> Provided under the [Software License Agreement for Beckhoff Software Products](https://www.beckhoff.com/media/downloads/general-terms-and-conditions/software_license_agreement_for_beckhoff_software_products.pdf)  
> Copyright (c) 2026 [Beckhoff Automation GmbH & Co. KG](https://www.beckhoff.com)  

## sACN References

Library based on protocol description ANSI E1.31 — 2018 (Document number: CP/2014-1009r6a)

[DMX] ANSI E1.11 Entertainment Technology – USITT DMX512-A Asynchronous Serial Digital Data Transmission Standard for controlling lighting equipment and accessories.

[ACN] ANSI E1.17 Entertainment Technology—Architecture for Control Networks
This standard is maintained by ESTA.

[ACN-DMP] ANSI E1.17 Architecture for Control Networks—Device Management Protocol
This standard is maintained by ESTA.

[RDM] ANSI E1.20 Entertainment Technology—Remote Device Management over DMX512 networks.
This standard is maintained by ESTA.

ESTA
630 Ninth Avenue, Suite 609
New York, NY 10036, USA. 
Phone: 1-212-244-1505
Fax: 1-212-244-1502
standards@esta.org
http://www.tsp.esta.org

## Library instalation

Steps:

1. Find Tc3_sACN.compiled-library in the Compiled Library folder.
2. Launch TwinCAT 3 XAE (Visual Studio).
From the main menu, choose PLC > Library Repository...
3. In the Library Repository dialog, click Install...
Browse to the Compiled Library folder and select Tc3_sACN.compiled-library.
4. Confirm the installation. The library should appear in the Installed list.
5. Add the library to your PLC project (once installed)
In your PLC project, right‑click References > Add library...
Search for Tc3_sACN and add it to the project.

Notes

If Windows has blocked the file (downloaded from the internet), open file Properties and click Unblock before installing.

## Protocol description 

sACN (Streaming Architecture for Control Networks) is a protocol designed for efficient, real-time communication of lighting control information over a network. It is widely used in the entertainment industry for controlling stage lighting, LED screens, and other visual effects. sACN allows multiple controllers to send DMX-style data streams to a variety of receivers, ensuring smooth transitions and synchronized effects. The protocol operates over standard IP networks, enabling scalable and flexible lighting setups. Its multicast capabilities allow for efficient data distribution, making it ideal for large-scale installations.

sACN transmits data over networks using UDP packets, leveraging multicast addressing to efficiently distribute information to multiple receivers. This method reduces network traffic and ensures synchronized data delivery across all connected devices. The data transmitted consists of DMX-style control information, organized into "universes," each capable of managing up to 512 channels. These universes allow for comprehensive control of lighting fixtures and other devices. Although UDP does not guarantee packet delivery, sACN is designed to handle occasional packet loss without disrupting real-time operations. Devices on the network need to be configured to join specific multicast groups, filtering and processing only the data relevant to their assigned universes.

![Fig. 1 sACN protocol concept](./resources/img/sACN_net.svg)

## Library concept 

The Tc3_sACN library provides a comprehensive solution for integrating sACN (Streaming Architecture for Control Networks) communication into PLC systems, enabling seamless control and monitoring of DMX data over Ethernet networks. It comprises three key functional blocks:

FB_sACN_Receiver: This block is responsible for receiving sACN data packets. It manages multicast communication, processes incoming packets, and handles multiple sources of DMX data. The receiver supports various merge rules, ensuring data from different sources is correctly integrated. It also tracks active sources and manages packets efficiently, making it suitable for dynamic lighting and stage control applications.

FB_sACN_Source: This block facilitates the transmission of sACN data packets. Users can configure DMX channels, source name, CID, universe, priority, and sending intervals. It manages UDP socket communication to send data packets, including sync and universe discovery packets, allowing precise control of lighting fixtures and other DMX-enabled devices.

FB_sACN_UDP_Communicator: Central to the library, this block handles UDP socket operations and network adapter interactions. It binds to network adapters, manages socket states, and ensures reliable multicast address handling for sACN receivers. The communicator oversees queues of sACN sources and receivers, orchestrating packet sending and receiving processes. It includes methods for updating network adapter information and managing socket events, ensuring robust and flexible communication in diverse network environments.

![Fig. 2 Library](./resources/img/lib_concept.svg)

Together, these blocks provide a powerful toolkit for implementing advanced sACN protocol functionality in industrial automation systems, optimizing DMX data handling for lighting and control applications.

## Project Description

This solution contains five sample PLC programs, each implemented as a subprogram that demonstrates a distinct aspect of the Tc3_sACN library. It is designed so only one sample runs at a time: a case statement selects the active subprogram based on the ActiveSample variable, while the others remain idle. Before running any sample, select the network interface in that sample’s code by setting the NetworkAdapterNum input of FB_sACN_UDP_Communicator. Switch ActiveSample to explore different scenarios without changing the project structure.

## CASE 01: Simple Send and Receive sACN Data

This example shows a compact end‑to‑end sACN workflow in one PLC program using three patterns. It sends universe 111 by directly writing the first 16 DMX channels to the sender’s DmxChannels, receives universe 222 and, on each new packet, copies the first 16 slots into an output array, and uses zero‑copy external array binding to send universe 333 and to receive/merge universe 444. The receiver on universe 444 employs the Highest Value Takes Precedence merge rule and writes merged values directly into a bound array. A single UDP communicator (selectable NIC) is shared by all function blocks; the Enable input gates activity, and isInited indicates that one‑time array binding is complete. Separate SourceName/CID, priorities, and send periods illustrate typical sender configuration.

## CASE 02: Single Communicator, Multiple Subprograms

Subprogram #2 demonstrates how a single FB_sACN_UDP_Communicator instance can be shared across multiple subprograms when only one communicator is allowed per network interface. The parent creates and configures the communicator (including NetworkAdapterNum) and passes it by interface to each child via the input sACN_ExternalCommunicator : Tc3_sACN.I_sAcnCommunicator. Transmission and reception are split into separate subprograms, both operating on the injected communicator instance. This pattern avoids global variables, preserves encapsulation, and cleanly composes modules around a shared resource.

## CASE 03: Single Communicator, Multiple Subprograms

This subprogram demonstrates how to integrate the sACN protocol with hardware DMX terminals. Terminal interaction is encapsulated in the IO folder subprograms. The example shows two integration techniques: zero‑copy binding of receivers and sources to external arrays for direct data flow, and explicit copying of slot values from received sACN packets into DMX terminal buffers (and vice versa). Yoy can use it as a template for routing and mirroring channel data between sACN universes and physical DMX lines.

## CASE 04: Idea for multiuniverse sACN source

Subprogram #4 models a multi‑universe sACN transmitter device. It runs multiple sACN Source instances in parallel—one per universe—while sharing a single FB_sACN_UDP_Communicator. 

## CASE 05: Idea for multiuniverse sACN source

Subprogram #5 shows how to extend the Tc3_sACN library’s function blocks to meet specialized requirements. As a concrete example, it implements per-address priority mode, adding support for producing and consuming per-slot priority data and handling merges that mix universe-level and per-slot priorities. The extended sender and receiver POUs are provided in the “SBSPM Extended POUs” folder and can be used as drop-in replacements for the standard blocks. This pattern keeps custom logic isolated from the base library while preserving interface compatibility.

## CASE 06: Idea for Tracking Events from Tc3_EventLogger

All functional blocks in this library write their messages to the TwinCAT Event Logger. When you need to react to these events in code, you can catch them directly in your application by inheriting from FB_sAcnLibEventsCatcher. This base component already implements the event subscription and dispatch logic, so the approach is context‑independent and requires no additional configuration. Subprogram #6 monitors the OnSACN_Receiver_Disabled and OnSACN_Receiver_Enabled events. It shows how to override the methods provided by FB_sAcnLibEventsCatcher and implement the actions your application should take. The base class defines these methods and wires the events for you; your overrides add the custom behavior while the catcher handles the rest.

## License

- [License](./LICENSE.md)