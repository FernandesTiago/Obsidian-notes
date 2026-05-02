The OSI Model is a model on how systems connect

Open System Interconnection Reference Model

there are 7 layers, the bottom being the Layer 1 

Layer 1 - Physical
Layer 2 - Data Link
Layer 3 - Network
Layer 4 - Transport
Layer 5 - Session
Layer 6 - Presentation
Layer 7 - Application

Layer 1 - Physical
- Signaling, Cabling, Connectors, fiber 
- signaling (wireless signal and signal going through cable)
- Not about protocols

Problem on physical layer might be bad cable, wireless interference, others

Layer 2 - Data Link
DLC (Data Link Control) protocols
- MAC (Media Access Control) address on Ethernet
- Fundamental layer of communication between two devices
- the "switching" layer
- (Frame, Mac EUI-48,-64(Extended Unique Identifier), Switch)

Layer 3 - Network
- the "Routing" layer
- IP (internet protocol)
- Fragment frames to traverse different networks (note: I think here is where wireshark works, those bunch 2 characters frames)
- (IP Address, subnet mask, Router, Packet)

Layer 4 - Transport
- the transport layer
- TCP (Transmission Control Protocol) - UDP (User Datagram Protocol)
- send the fragmented data and rebuild on the other side
- (TCP segment, UDP datagram)

Layer 5 - Session
- Communication management between devices
- (start, stop, restart)
- Control protocols, tunneling protocols (also layer 5)

Layer 6 - Presentation
- Character encoding
- application encryption and decryption
- can be combined with the application layer (Layer 7)
- (application encryption(SSL/TLS))

Layer 7 - Application
- What we see on screen
- HTTP/HTTPS, FTP, DNS, POP3

WIRESHARK is an amazing tool to understand the OSI Layers
