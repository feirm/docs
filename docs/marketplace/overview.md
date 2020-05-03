# Overview

The Feirm Marketplace originally started as a platform to allow users to exchange digital goods and services in exchange for cryptocurrency. The risk is eliminated by using trustless escrow that is handled by the blockchain, and traders can talk to each other in a secure, peer-to-peer fashion with end-to-end encrypted messages. The content of those messages cannot be decrypted unless the decryption key is voluntarily handed over to a moderator in the event of a dispute.

It is important to understand why the Feirm Marketplace isn't completely decentralised. This doesn't mean that the platform isn't secure from hacking or censorship. In order to ensure the Marketplace is accessible to everyone, there are three specific criteria that needs to be met.

## Criteria

### Portability
The Feirm Marketplace must work without additional software. The user having to install a dedicated application in order to use the Marketplace heavily affects its portablility as it limits the user to the same device. Almost all devices feature a web-browser, so why not make use of it?

### Accessibility
The Feirm Marketplace should work across all devices. Just like users check their Amazon orders on the go, the Marketplace needs to function on Desktops, Laptops, Tablets and Smartphones all at the same time.

### Asynchronous
Users shouldn't need to be online in order to accept a new trade offer or to receive an encrypted message. Encrypted information must be exchanged asynchronously. It is necessary to have somewhere reliable in order to store this data and allow the user to retrieve it at a later date.

## Decentralised Hybrid
As stated at the start, the Feirm Marketplace isn't totally decentralised. It relies on a set of centralised and decentralised components to form a suitable balance of security and usability. Just like how Signal, the end-to-end encrypted instant messenger, relies on a central server, the same applies for the Feirm Marketplace. As the technology evolves for decentralised applications over the coming years, the long-term goal is to decentralise more components which make up the Feirm Marketplace. 

There a three layers when it comes to the design of the Feirm Marketplace:

1. The centralised layer
2. The cryptography layer
3. The blockchain layer

## Centralised Layer
The centralised layer (API) is used to store and transport encrypted data and metadata required to make the Feirm Marketplace functional. It is made up entirely of the API, servers and databases. It has been designed in such a way that no sensitive information is disclosed to the centralised layer that could compromise the users security in the event of an attack.

**It is important to note that on this layer, no funds can be stolen in the event of an attack due to the nature of client-side encryption.**

What you can expect to take place on this layer is the following:

* Email notifications
* Product listings
* Suspension of user accounts
* Storage and transport of encrypted data

## Cryptography Layer
This layer is used to encrypt and verify sensitive information. It happens offline and inside the users web-browser. Actions here include signing and encrypting messages between users, creating digital signatures for authentication checks and decrypting encrypted data with private keys.

What you can expect to take place on this layer is the following:

* Diffie-Hellman key exchanges
* Encrypted messages
* Client-side web wallet

## Blockchain Layer
This layer involves any action to do with the blockchain. This could include forming transactions, submitting them to the network and also retrieving data from the network.

What you can expect to take place on this layer is the following:

* Escrow transactions (using Script)
