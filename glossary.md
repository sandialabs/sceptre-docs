## Acronyms

| *NAME* | *DEFINITION* |
| ------ | ------------ |
| AMS | Adaptive Multichannel Source |
| CLI | Command-Line Interface |
| COTS | Commercial Off the Shelf |
| CSV | Comma-Separated Value |
| DPT | Differential Pressure Transmitter |
| ELK | Elasticsearch, Logstash, Kibana |
| FEP | Front End Processor |
| GOOSE | Generic Object Oriented Substation Event |
| GPS | Global Positioning System |
| GUI | Graphical User Interface |
| HIL | Hardware-in-the-loop |
| HMI | Human Machine Interface |
| HTTP | HyperText Transport Protocol |
| ICS | Industrial Control System |
| IDS | Intrusion Detection System |
| IED | Intelligent Electronic Device |
| I/O | Input/Output |
| IP | Internet Protocol |
| JSON | JavaScript Object Notation |
| KVM | Kernel-based Virtual Machine |
| NIC | Network Interface Card |
| NTP | Network Time Protocol |
| OS | Operating System |
| OPC | OLE (Object Linking and Embedding) for Process Control |
| OVS | Open vSwitch |
| PCAP | Packet Capture |
| QEMU | Quick Emulator |
| RTU | Remote Terminal Unit |
| SCADA | Supervisory Control and Data Acquisition |
| SCEPTRE | SCEPTRE (not an acronym) |
| SCP | Secure Copy |
| SEL | Schweitzer Engineering Laboratories |
| SoH | State of Health |
| SSH | Secure Shell |
| TCP | Transmission Control Protocol |
| TDN | Trunked Data Network |
| UDP | User Datagram Protocol |
| VLAN | Virtual Local Area Network |
| VM | Virtual Machine |
| VNC | Virtual Network Computing |

## Terminology

| *NAME* | *DEFINITION* |
| ------ | ------------ |
| Admin Network | Network used to link together the headnode and all the computes nodes |
| Compute Node | A physical server that hosts virtual machines deployed in the environment |
| cURL | "see URL", a command line tool for transferring data using various protocols, e.g., HTTP, HTTPS, FTP, etc. |
| Environment | The running deployed system of virtual machines and network |
| Experiment | A specific instantiation of a topology that is deployed using phēnix |
| Log Forwarding Agent | A program that runs on all deployed VMs to gather, collect, and ship logging information associated with SCEPTRE process logs, operating system logs, network connection statistics, and VM performance statistics |
| Headnode | Command and control computer that runs phēnix and minimega and monitor compute nodes |
| Metadata | User configuration files included in an experiment topology |
| minimega | A tool for launching and managing virtual machines. It is the underlying VM mangement software using by SCEPTRE |
| openvswitch | A multilayer virtual switch designed to enable massive network automation through programmatic extension, while still supporting standard management interfaces and protocols (e.g. NetFlow, sFlow, IPFIX, RSPAN, CLI, LACP, 802.1ag) |
| phēnix | Orchestration package that creates, deploys, and maintain experiments |
| phēnix Application | A process or service that runs on the deployed virtual machines |
| SCEPTRE Provider | Simulation of a real world process that provides process data to SCEPTRE field devices |
| SCEPTRE | SCEPTRE can't be defined, it's a way of life! |
| Solver | The physical process simulation software that supplies data to the provider interface |
| Topology | Collection of network definition files including yaml and metadata files |
| Trunked Data Network | A trunk used to move all the data from the experiment between the various compute nodes in the cluster |
