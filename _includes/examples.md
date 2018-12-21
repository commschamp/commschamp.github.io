There are multiple available examples of protocol definitions that use 
[CommsDSL](https://github.com/arobenko/CommsDSL-Specification) schema files
and **commsdsl2comms** code generator to generate relevant code.

# Synthetic Demo Protocols
- [cc.demo1.commsdsl](https://github.com/arobenko/cc.demo1.commsdsl) - Demo 
protocol to demonstrate definition of various fields as well as simple protocol framing.
The **generated** code can be viewed at 
[cc.demo1.generated](https://github.com/arobenko/cc.demo1.generated).
- [cc.demo2.commsdsl](https://github.com/arobenko/cc.demo2.commsdsl) - Demo 
protocol to demonstrate protocol versioning, where every message reports protocol
version in its transport frame.
The **generated** code can be viewed at 
[cc.demo2.generated](https://github.com/arobenko/cc.demo2.generated).
- [cc.demo3.commsdsl](https://github.com/arobenko/cc.demo3.commsdsl) - Demo 
protocol to demonstrate protocol versioning, where one "connect" message reports 
protocol version for all the subsequent messages.
The **generated** code can be viewed at 
[cc.demo3.generated](https://github.com/arobenko/cc.demo3.generated).

Every synthetic demo protocol also provides 2 example applications: **server** and 
**client**, which exchange messages. 

# Real Life Open Protocols
- [cc.mqtt311.commsdsl](https://github.com/arobenko/cc.mqtt311.commsdsl) - 
Defines [MQTT v3.1.1](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.pdf)
protocol, where message ID shares
the same byte with extra flags, that may define existence of the fields in
message payload as well as influence how message needs to be processed. The protocol
definition also contains snippets of custom code that are injected by the
**commsdsl2comms** into the generated one.
The **generated** code can be viewed at 
[cc.mqtt311.generated](https://github.com/arobenko/cc.mqtt311.generated).
- [cc.mqtt5.commsdsl](https://github.com/arobenko/cc.mqtt5.commsdsl) - 
protocol. Similar to v3.1.1, but also adds a heterogeneous list of 
various properties.
The **generated** code can be viewed at 
[cc.mqtt5.generated](https://github.com/arobenko/cc.mqtt5.generated).
- [cc.mqttsn.commsdsl](https://github.com/arobenko/cc.mqttsn.commsdsl) - 
Defines [MQTT-SN](http://mqtt.org/2013/12/mqtt-for-sensor-networks-mqtt-sn) 
protocol, where field of remaining length of the message can 
have either 1 or 3 bytes length, depending on the value it contains.
The **generated** code can be viewed at 
[cc.mqttsn.generated](https://github.com/arobenko/cc.mqttsn.generated).
- [cc.ublox.commsdsl](https://github.com/arobenko/cc.ublox.commsdsl) - 
Defines **UBX** protocol used by various
[u-blox GPS receivers](https://www.u-blox.com/en/position-time). The protocol
itself is quite complex with hundreds of messages. It uses custom checksum
calculation algorithms and injects multiple code snippets to fix incorrect
or incomplete functionality available by default.
The **generated** code can be viewed at 
- [cc.ublox.generated](https://github.com/arobenko/cc.ublox.generated).

