# SocketHelper

<br>

# About

![thumbnail](./assets/thumbnail.png)

- UE plugin to handle TCP (client/server) and UDP (peer) socket communications
- Multi threaded implementation for each connection
- Async nodes to quickly create a tcp (client/server) or udp socket
- Send binary data (bytes) or text with specific encoding
- Get network informations about your machine or an ip address
- Use async blueprint functions to quickly setup a socket and establish a communication

<br>

# Setup

![Nodes](./assets/nodeshd.png)

1. [Get the plugin on the marketplace](https://www.unrealengine.com/marketplace/en-US/product/tcp-udp-socket-client-server-helper) and install the plugin for the engine version you wish to use
2. Create or open an unreal engine project with a supported version
3. In the editor, go to Edit/Plugins, search for the plugin, check the box to enable it and restart the editor
4. When a new plugin version is available, go to your Epic Games Launcher, under Unreal Engine/Library, below the engine version, you will find your installed plugins, find the plugin and click on update, then wait for it to finish and restart your editor

<br>

# Support

### Bugs/Issues

If you encounter issues with this plugin, you **should** report it, to do so, in the editor, go to Edit/Plugins, search for this plugin, click on the plugin support button, this will open your browser and navigate to the plugin issue form where you need to fill in all the relevant details about your issue, this will help me investigate and reproduce it on my own in order to fix it. Be precise and give as many details as you can. Once solved, a new plugin version will be submitted to the marketplace, update the plugin and you are good to go. **Due to epic marketplace limitations, I can only patch/update this plugin for the last 3 engine version, older engine versions will not be supported anymore.**

### Feature requests

If you want a new feature relevant to this plugin use case, you can submit a request in the [plugin marketplace question page](https://www.unrealengine.com/marketplace/en-US/product/tcp-udp-socket-client-server-helper/questions). I **may** add this new feature in a future plugin version.

<br>

# Documentation

_Screenshots may differ from the latest plugin version, some features may have evolved or have been removed if deprecated._

**ESocketError** is an enum used to enumerate all the errors that can occur

**ESocketTextEncoding** is an enum used to enumerate all the text encodings available

**ESocketHelperIpProtocol** is an enum used to enumerate all the ip protocols available

**FInfoAddr** is a struct used to provide address informations like Ip, Port, IpProtocol, SocketProtocol.

**FInfosAddr** is a struct used to provide addresses informations like CanonicalName, Addresses.

**FSocketHelperAddress** is a struct used to represent an internet address and its data like Endpoint, Ip, Port, IpProtocol. 

### Utility

![utility nodes](./assets/node1hd.png)

| Node | Inputs | Outputs | Note |
| ---- | ------ | ------- | ---- |
| IsSocketSupported | | Result(Bool) | Checks whether the socket subsystem is supported by the platform | 
| Decode | Bytes(Array(Byte)), Encoding(ESocketTextEncoding) | Result(String) | Decodes an array of bytes with a specific encoding into a string |
| Encode | Str(String), Encoding(ESocketTextEncoding) | Result(Array(Byte)) | Encodes a string with a specific encoding into an array of bytes |
| IsIpv4Address | Address(String) | Result(Bool) | Checks whether the string provided is an ipv4 address |
| IsIpv6Address | Address(String) | Result(Bool) | Checks whether the string provided is an ipv6 address |
| HasNetworkInterface | | Result(Bool) | Checks whether any network interfaces are available on the machine |
| GetLocalInterfaces | | Result(Array(String)) | Returns the network interfaces available on the machine |
| GetMachineHostName | | HostName(String), Result(Bool) | Returns the host name found for the machine |
| ResolveHostName | Host(String) | Success(Bool), HostIp(String) | DNS resolves an hostname into an IP |
| AddCachedHostName | HostName(String), HostIp(String) | Result(Bool) | Saves an hostname and its ip in the cache for fast resolve query |
| RemoveCachedHostName | HostName(String) | Result(Bool) | Removes an hostname and its ip from the cache |
| GetAddressInfos | HostName(String), Service(String) | Success(Bool), Result(FInfosAddr) | Finds the ips for an hostname and a specific service like smtp, www, ... |
| MakeAddressFromIpPort | Ip(String), Port(Int) | Result(FSocketHelperAddress) | Creates an address from an ip and port |
| MakeAddressFromEndpoint | Endpoint(String) | Result(FSocketHelperAddress) | Creates an address from an endpoint |

### TCP Client

![tcp client nodes](./assets/node3hd.png)

A tcp client connects to a tcp server and exchanges data using the transfer control protocol (connectionful exchange)

**FTcpSocketOptions** is a struct used to provide socket options like RemoteAddress, LocalAddress, Name, ReceiveBufferSize, SendBufferSize, TextEncoding. If "LocalAddress" is not assigned when opening connection, a random endpoint on your machine will be bound to establish the connection, you must provide a valid remote endpoint though.

**FTcpSocketResult** is a struct used to provide event data result like BytesMessage, TextMessage, BoundAddress, ClosedByClient, ErrorCode, ErrorReason, Error.

#### Traditional nodes

_Bind to the events before opening connection to receive them properly_

| Node | Inputs | Outputs | Note |
| ---- | ------ | ------- | ---- |
| CreateTcpSocketClient | | Result(UTCPClientHandler) | Initiate a tcp client handler, save the reference into a variable to avoid garbage collection |
| Open | Options(FTcpSocketOptions) | Result(Bool) | Tries to open a socket and connection with remote endpoint based on options provided |
| Close | | Result(Bool) | Tries to close the socket and connection with remote endpoint. Call this function to end communication and free memory resources |
| SendBytes | Data(Array(Byte)) | ByteSent(Int), Result(Bool) | Sends data to the remote endpoint and returns the amount of bytes sent |
| SendText | Data(String), TextEncoding(ESocketTextEncoding) | ByteSent(Int), Result(Bool) | Sends encoded text to the remote endpoint and returns the amount of bytes sent |
| IsConnected | | Result(Bool) | Checks whether this client is connected to a server |
| GetLocalAddress | | Result(FSocketHelperAddress) | Returns the current local endpoint bound for this connection |
| OnConnected | | BoundAddress(FSocketHelperAddress) | Triggered when the connection is established on the local bound endpoint |
| OnClosed | | ClosedByClient(Bool) | Triggered when the connection was closed by the remote or the client |
| OnError | | Code(Int), Reason(String), Error(ESocketError) | Triggered when an error occurs, reason contains additional information |
| OnTextMessage | | Message(String) | Triggered when a text message is received from remote |
| OnByteMessage | | Message(Array(Byte)) | Triggered when a bytes message is received from remote |

#### One node to rule them all (Async)

_These compact nodes return a handler to access the same feature as the traditional approach described above, calling these async nodes again won't create another socket unless you closed the previous one_

| Node | Inputs | Outputs | Note |
| ---- | ------ | ------- | ---- |
| AsyncTcpSocket | Options(FTcpSocketOptions) | OutHandler(UTCPClientHandler) | Initiate and opens a tcp socket with provided options |
| OnConnected | | Result(FTcpSocketResult) | Event triggered when connection is established |
| OnTextMessage | | Result(FTcpSocketResult) | Event triggered when a text message is received |
| OnBytesMessage | | Result(FTcpSocketResult) | Event triggered when a binary message is received |
| OnClosed | | Result(FTcpSocketResult) | Event triggered when connection is closed |
| OnError | | Result(FTcpSocketResult) | Event triggered when an error occurs, you can check the "ErrorReason" and "Error" properties to have more details about it |

### TCP Server

![tcp server nodes](./assets/node5hd.png)

A tcp server listens for incoming connections from tcp clients and communicates with them using the transfer control protocol (connectionful exchange)

**FTcpServerOptions** is a struct used to provide socket options like LocalAddress, ListenIntervalRate, ConnectionQueueSize, MaxClients, Name, ReceiveBufferSize, SendBufferSize, TextEncoding. A valid "LocalAddress" must be provided to start the server.

**FTcpServerResult** is a struct used to provide event data result like BoundAddress, Clients, ByteMessage, TextMessage, SenderAddress, ConnectedAddress, DisconnectedAddress, HasLostConnection, ErrorCode, ErrorReason, Error.

#### Traditional nodes

_Bind to the events before starting server to receive them properly_

| Node | Inputs | Outputs | Note |
| ---- | ------ | ------- | ---- |
| CreateTcpSocketServer | | Result(UTCPServerHandler) | Initiate a tcp server handler, save the reference into a variable to avoid garbage collection |
| Start | Options(FTcpServerOptions) | Result(Bool) | Starts a tcp server and listen for connections attempt |
| Stop | | Result(Bool) | Closes the communications with connected clients and free resources |
| Pause | | Result(Bool) | Stops listening for incoming connections, but still receives data from connected clients |
| Resume | | Result(Bool) | Resume listening for incoming connections |
| DisconnectClient | ClientAddress(FSocketHelperAddress) | Result(Bool) | Closes the connection with a specific client |
| SendText | Data(String), TextEncoding(ESocketTextEncoding) | ByteSent(Int), Result(Bool) | Sends text message to all connected clients |
| SendBytes | Data(Array(Byte)) | ByteSent(Int), Result(Bool) | Sends bytes message to all connected clients |
| SendTextTo | Data(String), ClientAddress(FSocketHelperAddress), TextEncoding(ESocketTextEncoding) | ByteSent(Int), Result(Bool) | Sends text message to a specific connected client |
| SendBytesTo | Data(Array(Byte)), ClientAddress(FSocketHelperAddress) | ByteSent(Int), Result(Bool) | Sends bytes message to a specific connected client |
| IsListening | | Result(Bool) | Returns whether the server is listening to incoming connection attempt |
| GetBoundAddress | | Result(FSocketHelperAddress) | Returns the local bound endpoint on which this server is listening |
| GetClientCount | | Result(Int) | Returns the number of clients connected to the server |
| GetClients | | Result(Array(FSocketHelperAddress)) | Returns the addresses of the clients connected to the server |
| OnStart | | BoundAddress(FSocketHelperAddress) | Triggered when the server is started and listens on a specific endpoint |
| OnStop | | | Triggered when the server is stopped |
| OnError | | Code(Int), Reason(String), EError(ESocketError) | Triggered when an error occurs, reason contains additional information |
| OnConnected | | ConnectedAddress(FSocketHelperAddress), Count(Int) | Triggered when a new client connects to the server |
| OnDisconnected | | DisconnectedAddress(FSocketHelperAddress), HasLostConnection(Bool), Count(Int) | Triggered when a client is disconnected from the server |
| OnTextMessage | | Message(String), SenderAddress(FSocketHelperAddress) | Triggered when a new text message is received from a client |
| OnByteMessage | | Message(Array(Byte)), SenderAddress(FSocketHelperAddress) | Triggered when a new byte message is received from a client |

#### One node to rule them all (Async)

_These compact nodes return a handler to access the same feature as the traditional approach described above, calling these async nodes again won't create another socket unless you closed the previous one_

| Node | Inputs | Outputs | Note |
| ---- | ------ | ------- | ---- |
| AsyncTcpServer | Options(FTcpServerOptions) | OutHandler(UTCPServerHandler) | Initiate and starts a tcp server with provided options |
| OnStart | | Result(FTcpServerResult) | Event triggered when server is started |
| OnConnected | | Result(FTcpServerResult) | Event triggered when a new client is connected |
| OnTextMessage | | Result(FTcpServerResult) | Event triggered when a text message is received |
| OnByteMessage | | Result(FTcpServerResult) | Event triggered when a binary message is received |
| OnDisconnected | | Result(FTcpServerResult) | Event triggered when a client is disconnected |
| OnStop | | Result(FTcpServerResult) | Event triggered when server is stopped |
| OnError | | Result(FTcpServerResult) | Event triggered when an error occurs, you can check the "ErrorReason" and "Error" properties to have more details about it |

### UDP Socket

![udp peer nodes](./assets/node4hd.png)

An udp peer listens for incoming messages from other udp peers and communicates with them using the user datagram protocol (connectionless exchange)

**FUdpSocketOptions** is a struct used to provide socket options like ListenAddress, Name, ReceiveBufferSize, SendBufferSize, TextEncoding. If you do not provide a "ListenAddress", you won't be receiving message, but will still be able to send them.   

**FUdpSocketResult** is a struct used to provide event data result like BytesMessage, TextMessage, SenderAddress, IsBound, BoundAddress, ErrorCode, ErrorReason, Error.

#### Traditional nodes

_Bind to the events before opening socket to receive them properly_

| Node | Inputs | Outputs | Note |
| ---- | ------ | ------- | ---- |
| CreateUdpSocket | | Result(UUDPSocketHandler) | Initiate an udp socket handler, save the reference into a variable to avoid garbage collection |
| Open | Options(FUdpSocketOptions) | Result(Bool) | Tries to create and eventually bind the socket to a local endpoint |
| Close | | Result(Bool) | Tries to close the socket |
| SendBytes | Data(Array(Byte)), RemoteAddress(FSocketHelperAddress) | ByteSent(Int), Result(Bool) | Sends data to the remote endpoint and returns the amount of bytes sent |
| SendText | Data(String), RemoteAddress(FSocketHelperAddress), Encoding(ESocketTextEncoding) | ByteSent(Int), Result(Bool) | Sends encoded text to the remote endpoint and returns the amount of bytes sent |
| IsListening | | Result(Bool) | Returns whether the socket is listening for incoming messages |
| IsRunning | | Result(Bool) | Returns whether the socket is valid |
| OnConnected | | IsBound(Bool), BoundAddress(FSocketHelperAddress) | Triggered when the socket is created and eventually bound to a local endpoint for incoming messages |
| OnClosed | | | Triggered when the socket was closed |
| OnError | | Code(Int), Reason(String), Error(ESocketError) | Triggered when an error occurs, reason contains additional information |
| OnTextMessage | | Message(String), SenderAddress(FSocketHelperAddress) | Triggered when a new text message is received from a peer |
| OnByteMessage | | Message(Array(Byte)), SenderAddress(FSocketHelperAddress) | Triggered when a new byte message is received from a peer |

#### One node to rule them all (Async)

_These compact nodes return a handler to access the same feature as the traditional approach described above, calling these async nodes again won't create another socket unless you closed the previous one_

| Node | Inputs | Outputs | Note |
| ---- | ------ | ------- | ---- |
| AsyncUdpSocket | Options(FUdpSocketOptions) | OutHandler(UUDPSocketHandler) | Initiate and creates a udp socket with provided options |
| OnConnected | | Result(FUdpSocketResult) | Event triggered when socket is created and active |
| OnTextMessage | | Result(FUdpSocketResult) | Event triggered when a text message is received from a peer |
| OnBytesMessage | | Result(FUdpSocketResult) | Event triggered when a binary message is received from a peer |
| OnClosed | | Result(FUdpSocketResult) | Event triggered when socket is closed |
| OnError | | Result(FUdpSocketResult) | Event triggered when an error occurs, you can check the "ErrorReason" and "Error" properties to have more details about it |

<br>

# Demos

[Demo TCP client](https://blueprintue.com/blueprint/gjhsl1lv/)

[Demo TCP server](https://blueprintue.com/blueprint/0ccxi1_0/)

[Demo UDP peer](https://blueprintue.com/blueprint/ksj8awk8/)