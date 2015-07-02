Named WebSockets
===

#### Dynamic binding, peer management and local service discovery for WebSockets ####

Named WebSockets is a bootstrap mechanism for creating, binding and connecting WebSocket peers together on a local machine or on a local network that share the same *channel name*. Named WebSockets can be created from both web applications and native applications to create communications bridges between and among themselves.

A web page or application can create a new Named WebSocket by choosing an appropriate *channel type* (`local` or `network`) and *channel name* (any alphanumeric name) via one of the available [Named WebSocket interfaces](#named-websocket-interfaces). When other web pages or applications create their own Named WebSocket connection with the same *type* and *name* then it will act as a full-duplex broadcast channel between all other connected Named WebSocket peers.

![Named WebSockets](https://raw.githubusercontent.com/namedwebsockets/namedwebsockets/images/networkwebsockets_diagram.png "Named WebSockets")

Named WebSockets are useful in a variety of collaborative local device and local network scenarios:

* Discover matching peer services on the local device and/or the local network.
* Create full-duplex communications channels between native applications and web applications.
* Create full-duplex communication channels between web pages hosted on different domains.
* Create initial local session signalling channels for establishing P2P sessions (via e.g. [WebRTC](#examples)).
* Create low latency local network multiplayer signalling channels for games.
* Enable collaborative editing, sharing and other forms of communication between web pages and applications on the local device and/or the local network.

This repository contains the proof-of-concept _Named WebSockets Proxy_, written in Go, currently required to use Named WebSockets. You can [download and run this proxy](#getting-started) on your own machine and start experimenting with Named WebSockets following the instructions provided below.

Once you have a Named WebSockets Proxy up and running on the local machine then you are ready to [create and share your own local/network Named WebSockets](#named-websocket-interfaces). A number of [Named WebSocket examples](#examples) are also provided to help get you started.

### Getting started

To create and share Named WebSockets currently requires a Named WebSockets Proxy to be running on each local machine that wants to participate in the network.

#### Download a pre-built binary

You can download and run the latest pre-built version of the Named WebSockets Proxy from the [downloads page](https://github.com/namedwebsockets/namedwebsockets/releases).

[Go to the latest downloads page](https://github.com/namedwebsockets/namedwebsockets/releases)

#### Build from source

Optionally you can run a Named WebSockets Proxy from the latest source files contained in this repository with the following instructions:

1. [Install go](http://golang.org/doc/install).

2. Download this repository and its dependencies using `go get`:

        go get github.com/namedwebsockets/cmd/namedwebsockets

3. Locate and change directory to the newly downloaded repository:

        cd `go list -f '{{.Dir}}' github.com/namedwebsockets/cmd/namedwebsockets`

4. Run your Named WebSockets Proxy:

        go run run.go

At this point your Named WebSockets Proxy should be up and ready for usage at `localhost:9009`!

You can now start using your Named WebSockets Proxy via any of the [Named WebSocket Proxy Interfaces](#named-websocket-interfaces) described below.

### Named WebSocket Interfaces

#### Local HTTP Test Console

Once a Named WebSockets Proxy is up and running, you can access a test console and play around with both `local` and `network` Named WebSockets at `http://localhost:9009/console`.

#### JavaScript Interfaces

The [Named WebSockets JavaScript polyfill library](https://github.com/namedwebsockets/namedwebsockets/blob/master/lib/namedwebsockets.js) exposes two new JavaScript interfaces on the root global object (e.g. `window`) for your convenience as follows:

* `LocalWebSocket` for creating/binding named websockets to share on the local machine only.
* `NetworkWebSocket` for creating/binding named websockets to share both on the local machine and the local network.

You must include the polyfill file in your own projects to create these JavaScript interfaces. Assuming your [Named WebSockets JavaScript polyfill](https://github.com/namedwebsockets/namedwebsockets/blob/master/lib/namedwebsockets.js) is located at `lib/namedwebsockets.js` then that can be done in an HTML document as follows:

    <script src="lib/namedwebsockets.js"></script>

You can create a new `LocalWebSocket` connection object via the JavaScript polyfill as follows:

    var localWS = new LocalWebSocket("myChannelName");
    // Now do something with `localWS` (it is a WebSocket object so use accordingly)

You can create a new `NetworkWebSocket` connection object via the JavaScript polyfill as follows:

    var networkWS = new NetworkWebSocket("myChannelName");
    // Now do something with `networkWS` (it is a WebSocket object so use accordingly)

When any other client connects to a `local` or `network` websocket endpoint named `myChannelName` then your websocket connections will be automatically linked to one another. Note that `local` and `network` based websocket connections are entirely seperate entities even if they happen to share the same channel name.

You now have a full-duplex WebSocket channel to use for communication between each peer connected to the same channel name with the same channel type!

#### WebSocket Interfaces

Devices and services running on the local machine can register and use Named WebSockets without needing to use the JavaScript API. Thus, we can connect up other applications and devices sitting in the local network such as TVs, Set-Top Boxes, Fridges, Home Automation Systems (assuming they run their own Named WebSockets proxy client also).

To create a new `local` Named WebSocket channel from anywhere on the local machine, simply open a new WebSocket to `ws://localhost:9009/local/<channelName>/<peer_id>`, where `channelName` is the name of the channel you want to create and use and `peer_id` is a new random integer to identify the new peer you are registering on this Named WebSocket.

To create a new `network` Named WebSocket channel from anywhere on the local machine, simply open a new WebSocket to `ws://localhost:9009/network/<channelName>/<peer_id>`, where `channelName` is the name of the channel you want to create and use and `peer_id` is a new random integer to identify the new peer you are registering on this Named WebSocket. In addition to `local` Named WebSocket channels, this type of Named WebSocket will be advertised in your local network and other Named WebSocket Proxies running in the network will connect to your broadcasted web socket interface.

### Examples

Some example services built with Named WebSockets:

* [Chat example](https://github.com/namedwebsockets/namedwebsockets/tree/master/examples/chat)
* [PubSub example](https://github.com/namedwebsockets/namedwebsockets/tree/master/examples/pubsub)
* [WebRTC example](https://github.com/namedwebsockets/namedwebsockets/tree/master/examples/webrtc)

### Discovery and service advertisement mechanism

Named Websockets uses multicast DNS-SD (i.e. Zeroconf/Bonjour) to discover `network` channels on the local network. The proxy connections that result from this process between Named WebSocket Proxies are used to transport WebSocket messages between different network peers using the same *channel name* in the local network.

Named WebSocket channels all use the DNS-SD service type `_ws._tcp` with a unique channel name in the form `<channelName>[<UID>]` (e.g. `myChannel[2049847123]`) and include a `path` attribute in the TXT record corresponding to the WebSocket's absolute endpoint path (e.g. `path=/network/myChannel`). From these advertisements it is possible to resolve Named WebSocket endpoint URLs that remote proxies can use to connect with each other.

When a new `network` WebSocket is created then the local Named WebSockets Proxy must notify (i.e. 'ping') all other Named WebSocket Proxies in the local network about this newly created channel via the DNS-SD broadcast.

When a remote Named WebSockets Proxy detects a new `network` on the multicast DNS-SD port then it immediately establishes a connection to that Named WebSocket's URL and then creates its own new `network` WebSocket to advertise back (i.e. 'pong') to other peers.

These processes repeats on all Named WebSocket Proxies whenever they receive a previously unseen 'ping' or 'pong' Named WebSocket advertisement broadcast.

### Feedback

If you find any bugs or issues please report them on the [Named WebSockets Issue Tracker](https://github.com/namedwebsockets/namedwebsockets/issues).

If you would like to contribute to this project please consider [forking this repo](https://github.com/namedwebsockets/namedwebsockets/fork), making your changes and then creating a new [Pull Request](https://github.com/namedwebsockets/namedwebsockets/pulls) back to the main code repository.

### License

MIT.
