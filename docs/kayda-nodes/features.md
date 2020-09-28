# Data Storage

Typically, when it comes to storing vast amounts of data on a blockchain, it is a costly operation due to the transaction fees that have to be paid. Additionally, even if we wanted to store a large amount of data, we are limited to a maximum size of 80 bytes if the `OP_RETURN` script is used within a transaction. For modern day applications such as instant messaging and files sharing, this simply isn't enough.

To combat this issue, every Kayda node has the ability to store data. At this moment in time, only textual data can be stored. The data field on each message object can hold a maximum of 1MB, which is plenty for modern day applications using the Kayda network as an ephemeral data store. Furthermore, to keep the Kayda network spam-free, every message has a TTL of 3 days from the time it is received at an individual node.

Both of the values above (data limit and TTL) can easily be modified in the future if needed without affecting the underlying Feirm cryptocurrency network.