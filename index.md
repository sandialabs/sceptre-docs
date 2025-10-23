
![](img/sceptre.gif)

# Overview

SCEPTRE is a modeling, simulation, and test bedding environment for industrial control systems ([ICSs](glossary.md#ics)) that bridges the gap between control system models and process simulation. Tools and techniques exist for simulating and emulating control system field devices but results from the security analysis that these tools currently support are limited because the physical processes being controlled are not included. SCEPTRE leverages proven technologies and techniques to intertwine the device and process simulations, providing an integrated system capable of representing realistic responses in the physical process as events occur in the control system and vice versa.

SCEPTRE is comprised of simulated control system devices, such as remote terminal units (RTUs), programmable logic controllers (PLCs) and protection relays, and simulated processes, such as electric power transmission systems, refinery processes, and pipelines. The simulated control system devices can communicate over Internet Protocol (IP) networks using standard Supervisory Control and Data Acquisition (SCADA) protocols like Modbus, DNP3, and others. SCEPTRE also includes support for hardware-in-the-loop, wherein real field devices under study (i.e., a specific model of PLC) can be connected to and interact with the physical process being simulated.

The SCEPTRE platform provides a means for creating large-scale control system test environments suitable for cyber-physical security experiments. Leveraging modeling, simulation, and test bedding techniques, the test environments can be scripted to suite each experiment as necessary, are repeatable, and are much cheaper to construct than real or even lab-scale test environments. The standards-based SCADA protocols in the simulated field devices enable the use of 3rd party ICS and cyber security testing applications and supports the use of simulated and emulated network environments.

For a broader description of SCEPTRE, please read our white-paper,  [SCEPTRE: A Cyber-Physical Emulation Capability](https://www.osti.gov/biblio/2999077).

To get started, take a look at our [Quick Start Guide](quick-start.md).

# Features

- Easily create, configure, and deploy an experiment from a user-friendly web interface
- Start and stop an experiment with a single button
- Industry standard Supervisory Control and Data Acquisition (SCADA) applications and protocols
- Sandia-developed simulated and emulated Industrial Control System (ICS) devices
- High-fidelity end-process simulation
- Software defined networking
- Support for hardware-in-the-loop

# Components

SCEPTRE consists of several different components/code bases:

- [sceptre-phenix](https://github.com/sandialabs/sceptre-phenix): Phēnix is an orchestrator that manages the creation, configuration, and deployment of modeling and simulation experiments. The phenix documentation is here: [phenix.sceptre.dev](https://phenix.sceptre.dev/latest/)
- [sceptre-phenix-apps](https://github.com/sandialabs/sceptre-phenix-apps): Phēnix apps contains user applications written to work with phēnix. These applications add extra "stuff" to experiments. Most notable of apps is the "sceptre" app which add the ICS flavor to an experiment.
- [sceptre-phenix-topologies](https://github.com/sandialabs/sceptre-phenix-topologies): A topology is a collection of network definition files that define a specific model. Files contained in this repository contain some existing user created topologies.
- [sceptre-phenix-images](https://github.com/sandialabs/sceptre-phenix-images): phēnix image is a tool for quickly creating vm images with debian-based OSes. It is a wrapper of the vmdb2 tool which, in turn, is a wrapper of qemu-img, parted, kpartx, and debootstrap. Files contained in this repository can be used to create vm images for use in different topologies.
- [sceptre-bennu](https://github.com/sandialabs/sceptre-bennu): bennu is part of the SCEPTRE codebase that is responsible for modeling and simulation of ICS/SCADA devices and their communication protocols.

# Copyright

Copyright 2023 National Technology & Engineering Solutions of Sandia, LLC (NTESS). Under the terms of Contract DE-NA0003525 with NTESS, the U.S. Government retains certain rights in this software. SAND2023-13941O
