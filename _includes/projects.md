# Core Projects

| Repository | Details |
|----------------------|
|[COMMS Library](https://github.com/commschamp/comms)|The core component of the ecosystem, allows protocol definition to be simple, declarative statements.|
|[CommsDSL Specification](https://github.com/commschamp/CommsDSL-Specification)| The [CommsDSL](https://commschamp.github.io/commsdsl_spec) specification.|
|[Code Generators](https://github.com/commschamp/commsdsl)| [CommsDSL](https://commschamp.github.io/commsdsl_spec) schemas parsing and code generation tools.|
|[Tutorial](https://github.com/commschamp/cc_tutorial)|The official tutorial on how to define protocol, generate and use protocol definition code.|

.

# Synthetic Demo Protocols

| Main Repository | Generated Code | Details |
|--------------------------------------------|
|[cc.demo1.commsdsl](https://github.com/commschamp/cc.demo1.commsdsl)|[cc.demo1.generated](https://github.com/commschamp/cc.demo1.generated)|Demo protocol to demonstrate definition of various fields as well as simple protocol framing.|
|[cc.demo2.commsdsl](https://github.com/commschamp/cc.demo2.commsdsl)|[cc.demo2.generated](https://github.com/commschamp/cc.demo2.generated)|Demo protocol to demonstrate protocol versioning, where every message reports protocol version in its transport frame.|
|[cc.demo3.commsdsl](https://github.com/commschamp/cc.demo3.commsdsl)|[cc.demo3.generated](https://github.com/commschamp/cc.demo3.generated)|Demo protocol to demonstrate protocol versioning, where one "connect" message reports protocol version for all the subsequent messages.|

.

# Real Life Open Protocols


| Main Repository | Generated Code | Details |
|--------------------------------------------|
|[cc.mqtt311.commsdsl](https://github.com/commschamp/cc.mqtt311.commsdsl)|[cc.mqtt311.generated](https://github.com/commschamp/cc.mqtt311.generated)|Defines [MQTT v3.1.1](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.pdf) protocol, where message ID shares the same byte with extra flags, that may define existence of the fields in message payload as well as influence how message needs to be processed.|
|[cc.mqtt5.commsdsl](https://github.com/commschamp/cc.mqtt5.commsdsl)|[cc.mqtt5.generated](https://github.com/commschamp/cc.mqtt5.generated)|Defined [MQTT v5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html) protocol. Similar to **v3.1.1**, but also adds a heterogeneous list of various properties.|
|[cc.mqttsn.commsdsl](https://github.com/commschamp/cc.mqttsn.commsdsl)|[cc.mqttsn.generated](https://github.com/commschamp/cc.mqttsn.generated)|Defines [MQTT-SN](https://www.oasis-open.org/committees/download.php/66091/MQTT-SN_spec_v1.2.pdf) protocol, where field of remaining length of the message can have either 1 or 3 bytes length, depending on the value it contains.|
|[cc.ublox.commsdsl](https://github.com/commschamp/cc.ublox.commsdsl)|[cc.ublox.generated](https://github.com/commschamp/cc.ublox.generated)|Defines **UBX** protocol used by various [u-blox GPS receivers](https://www.u-blox.com/en/position-time). The protocol itself is quite complex with hundreds of messages. It uses custom checksum calculation algorithms and injects multiple code snippets to fix incorrect or incomplete functionality available by default.|

.

# Binary Data Encoding / Decoding

| Main Repository | Generated Code | Details |
|--------------------------------------------|
|[cc.asn1.commsdsl](https://github.com/commschamp/cc.asn1.commsdsl)||Provides [CommsDSL](https://commschamp.github.io/commsdsl_spec) schema definition to allow [ASN.1](https://en.wikipedia.org/wiki/ASN.1) encoding.|
|[cc.x509.commsdsl](https://github.com/commschamp/cc.x509.commsdsl)|[cc.x509.generated](https://github.com/commschamp/cc.x509.generated)| Provides definition of the [X.509](https://datatracker.ietf.org/doc/html/rfc5280) public key infrastructure certificate.|

.

# Protocol Handling Libraries

| Repository | Details |
|----------------------|
|[cc.mqttsn.libs](https://github.com/commschamp/cc.mqttsn.libs)|[MQTT-SN](https://www.oasis-open.org/committees/download.php/66091/MQTT-SN_spec_v1.2.pdf) client and gateway libraries.|
|[cc.mqtt5.libs](https://github.com/commschamp/cc.mqtt5.libs)|[MQTT5](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html) client library.|

.

# Tools

| Repository | Details |
|----------------------|
|[CommsChampion Tools](https://github.com/commschamp/cc_tools_qt)| Plugin based tools to visualize protocol definition and debug exchange of the protocol messages.|
|[cc.demo1_protocol.cc_tools_plugin](https://github.com/commschamp/cc.demo1_protocol.cc_tools_plugin)| The [demo1](https://github.com/commschamp/cc.demo1.commsdsl) synthetic **protocol** plugin.|
|[cc.demo2_protocol.cc_tools_plugin](https://github.com/commschamp/cc.demo2_protocol.cc_tools_plugin)| The [demo2](https://github.com/commschamp/cc.demo2.commsdsl) synthetic **protocol** plugin.|
|[cc.demo3_protocol.cc_tools_plugin](https://github.com/commschamp/cc.demo3_protocol.cc_tools_plugin)| The [demo3](https://github.com/commschamp/cc.demo3.commsdsl) synthetic **protocol** plugin.|
|[cc.mqtt311_protocol.cc_tools_plugin](https://github.com/commschamp/cc.mqtt311_protocol.cc_tools_plugin)| The [MQTT v3.1.1](https://github.com/commschamp/cc.mqtt311.commsdsl) **protocol** plugin.|
|[cc.mqttsn_protocol.cc_tools_plugin](https://github.com/commschamp/cc.mqttsn_protocol.cc_tools_plugin)| The [MQTT-SN](https://github.com/commschamp/cc.mqttsn.commsdsl) **protocol** plugin.|
|[cc.mqtt5_protocol.cc_tools_plugin](https://github.com/commschamp/cc.mqtt5_protocol.cc_tools_plugin)| The [MQTT v5](https://github.com/commschamp/cc.mqtt5.commsdsl) **protocol** plugin.|
|[cc.ublox_protocol.cc_tools_plugin](https://github.com/commschamp/cc.mqttsn_protocol.cc_tools_plugin)| The [U-blox](https://github.com/commschamp/cc.mqttsn.commsdsl) (UBX) **protocol** plugin.|
|[cc.mqtt5_client_filter.cc_tools_plugin](https://github.com/commschamp/cc.mqtt5_client_filter.cc_tools_plugin)| The MQTT v5 client **filter** plugin.|

.

# Package Managers and Build Systems

| Repository | Details |
|----------------------|
|[meta-commschamp](https://github.com/commschamp/meta-commschamp)|CommsChampion Ecosystem recipes provided as a [layer](https://docs.yoctoproject.org/bsp-guide/bsp.html) of the [yocto project](https://www.yoctoproject.org/).|
|[cc.buildroot](https://github.com/commschamp/cc.buildroot)|CommsChampion Ecosystem packages as an [external customization](https://buildroot.org/downloads/manual/manual.html#outside-br-custom) of the main [buildroot](https://buildroot.org/) build tree.|
|[cc.vcpkg](https://github.com/commschamp/cc.vcpkg)| CommsChampion Ecosystem packages as a [ports overlay](https://github.com/microsoft/vcpkg/blob/master/docs/specifications/ports-overlay.md) directory of the [vcpkg](https://github.com/microsoft/vcpkg) package manager.|
|[cc.cmake](https://github.com/commschamp/cc.cmake)| CommsChampion Ecosystem projects bundled into a single CMake one. Expected to be used in a signle [ExternalProject_Add()](https://cmake.org/cmake/help/latest/module/ExternalProject.html) invocation.|

.
