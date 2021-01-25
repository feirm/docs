# Overview

The Feirm Platform was originally developed as a Marketplace that allows users to exchange digital goods and services in exchange for cryptocurrency. The idea was to utilise the transaction scripting language known as Script to construct trustless escrow transactions.

Additionally, the Marketplace required some form of messaging functionality between users. This resulted in the need for encrypted messages - by default, all messages should be end-to-end encrypted. The only time where the encryption key for a conversation should be handed over is in the event of a dispute. Either user in a conversation must voluntarily hand over this key so that moderators can decrypt the messages and resolve any dispute between the two parties.

Since the idea of the Marketplace came about, we have expanded the scope of what we want the project to offer. Instead of providing a Marketplace, we wanted to provide an entire ecosyste - hence the Feirm Platform was born.

## Criteria
Before we got started with the Platform, it was important to lay out some criteria which we wanted to follow. In order for the Platform to be a success in our eyes, it must meet the following criteria.

### Portability
The Feirm Platform must work without requiring the user to install any additional software. If the user has to install a dedicated application to access and use the Feirm Platform, then the user is limited to a specific device and all portability aspects are lost. The Platform was developed as a client-side web application because the majority of all devices today have a web browser installed by default. It made sense to use what is already available to the majority.

### Availability
The Feirm Platform should work across all devices. This is where choosing to develop the Platform as a web application came in handy for us. The web application has been designed to be usable on Laptops, Tablets and Smartphones.

### Asynchronicity
Platform users shouldn't need to be online in order to accept a new trade offer or to receive an encrypted message. This is a common problem with traditional peer-to-peer applications - both parties need to be online in order to communicate. Instead, the Platform exchanges encrypted data asynchronously. This does require a level of trust from us though to store the encrypted data whilst a user is offline.

## Decentralised Hybrid
It should be made clear that at the moment, the Feirm Platform is not completely decentralised. It relies on a set of centralised and decentralised components (both maintained by the Feirm project) to provide a suitable balance of security and usability. Just like how Signal, the end-to-end encrypted instant messenger, relies on a central server to deliver encrypted messages to their recipients, the same applies for the Feirm Platform.

There a three layers when it comes to the design of the Feirm Platform:

1. The centralised layer
2. The cryptography layer
3. The blockchain layer

## Centralised Layer
The centralised layer (API) is used to store and transport encrypted data and metadata required to make the Feirm Platform functional and asynchronous. It is made up entirely of the API, caching servers and databases. This layer has been designed in such a way that no identifiable information could compromise the user in the event of a breach or attack due to the encryption used client-side and also the type of data which we store about a user account.

What you can expect to take place on this layer is the following:

* Account authentication - signing up and logging in.
* Storage and transport of encrypted data - listings, contact data and user preferences.

## Cryptography Layer
This layer is used to encrypt, decrypt and verify sensitive information. It happens client-side and inside the users web-browser. Actions here include encrypting messages between users, creating digital signatures for authentication processes and decrypting data with private keys.

What you can expect to take place on this layer is the following:

* Diffie-Hellman key exchanges - to derive a shared key between 2 users without revealing private keys.
* Encrypted messages - encryption, decryption and signing.
* Web wallet - generating a 24-word mnemonic used as the seed for a multi-asset wallet.

## Blockchain Layer
This layer involves any action to do with the blockchain. This could include creating transactions, submitting them to the network and also retrieving data from the network.

What you can expect to take place on this layer is the following:

* Escrow transactions (using Script).
