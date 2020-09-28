# Technical Overview
On the Kayda network, every node communications over the WebSocket protocol with the data payload being in JSON. The reason behind choosing these protocols is that they have solid implementations in other programming languages. If a developer wishes to use the Kayda network as a temporary data store for their application, our goal is to make that easy. The majority of popular browsers have native support for WebSockets, meaning that an application could directly establish a connection to a Kayda node.

The Kayda service daemon is currently written in the Go programming language. Due to its high performance, cross-platform ability and vast standard library, it was a great fit for our goal of keeping the Kayda network lightweight and fast.

# Configuration
Each Kayda node can be configured via a configuration file which is loaded upon every launch. The file makes use of the TOML format due to it being easy for users to understand. Here are the defaults in place for every configuration at the moment.

```toml
[protocol]
port = 44767
peers = [
    "kayda-service1.feirm.com",
    "kayda-service2.feirm.com",
    "kayda-service2.feirm.com",
]
```

By default, every Kayda node must have port `44767 TCP` open for WebSocket communication, as this is how nodes communicate with each other.

The Feirm project currently operate three Kayda service nodes which are used to bootstrap any new nodes who join the network. However, if you wish to bootstrap using other peers, you can do so simply by updating the configuration file.