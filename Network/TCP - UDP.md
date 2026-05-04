Layer 4 
TCP = With handshake
UDP = Faster with no handshake (send and trust)
5-tuple (protocol, Source IP, Source Port, Destination IP, Destination Port)

**TCP structure**

TCP carries a ton of metadata:
Source Port
Destination Port
Sequence Number
Acknowledgement Number
Flags: SYN | ACK | FIN | RST | PSH...
Window Size
(DATA) <-- payload

no need to remember it all, but its to understand that it carries a lot of info and that makes it heavy and slow, but also make it to be more trustful
(~20/80 bytes header)

**UDP is the opposite**

Source Port
Destination Port
Length 
Checksum
(DATA)
(8 bytes header)

being smaller in size makes it really fast.

**TCP flags**

TCP flags are bits that turn on and off 

SYN - "open connection" (synchronize)
ACK - "received your package" (acknowledge)
FIN - "close connetion" (finish)
RST - "reset connection" (reset)
PSH - "send immediately"
URG - "set as urgent" (rarely used)

so, 
SYN = first handshake package
SYN + ACK = second handshake package
ACK = third package
FIN + ACK = closing + last package

Sequence number (seq) = numbers each byte sent
acknowledgement number (ack) = "next byte I wanna receive"
ack=5001 it's like saying: "saw until byte 5000, send 5001 now"

**Graceful and Abrupt closing**

**Graceful (4-way FIN)**
client sends to server that will stop connection
server sends a okay
server sends to client that will stop connection
client sends a okay

**Abrupt  (RST)**
If anything goes wrong any side sends a RST and the connection dies instantly

(TCP RST attack) when a attacker sends a fake RST to down someone's connection

**TCP connection states**
LISTEN - server waiting connection
SYN-SENT - client sent a SYN, waiting SYN+ACK
SYN-RECV - server received SYN and sent SYN+ACK, waiting ACK
ESTABLISHED - connection working
FIN-WAIT-1 - sent FIN, waiting ACK
FIN-WAIT-2 - received ACK from my FIN, waiting other FIN
TIME-WAIT - closed connection (kernel keeps opening for about 60s)
CLOSE-WAIT - other side want to close, waiting my connection to close

All of it is important to debug and understanding connections
bunch of CLOSE-WAIT is a sign of bug
bunch of TIME-WAIT is normal

**When is UDP used**

DNS - 1 packet query, if lost just send another, low latency
NTP - Hour sync, small packet
VoIP - real time audio, it's better to lag them to delay
Online gaming - same thing
live streaming - same thing
Wireguard - TCP in TCP cause "TCP meltdown (study)"
DHCP - Client doesn't have IP so it can't establish a connection

TPC meltdown: performance issues due excessive re-transmissions and delays. 
happens because both TVP connections try to manage the packet loss independently causing a feedback loop. 

**When to use TCP**
Web (HTTPS) - full page has to reach
SSH - byte-per-byte
Email (SMTP/IMAP) - full message with no errors
Database queries - SQL can't lose caracters
File transfer - byte lost = corrupted file

**TCP vs UDP** 
basically TCP is more reliable but also slower with more bytes in the header and handshake, while UDP isn't as reliable it have and advantage in real-time events or low packet cost transmissions due to being faster (less 8 bytes header) and without the handshake it doesn't make the connection delay.

**QUIC - our generation TCP**

QUIC is a modern protocol developed by Google that runs on top of UDP but it implements the reliability almost like the TCP inside the payload, with obligatory cryptography and a way faster handshake

HTTP/3 (HTTP newest version) uses QUIC to access google services and Cloudflare, I can see if my browser is using QUIC by checking Wireshark.
