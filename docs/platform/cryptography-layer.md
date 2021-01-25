# Cryptography Layer

## Introduction
The majority of the cryptography for the Feirm Platform has been made possible thanks to the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API). This API allows web applications using JavaScript to perform cryptographic operations such as encryption, hashing and verifying digital signatures.

The Feirm Platform has made use of these cryptographic operations in order to secure information and enable end-to-end encrypted messaging between users on the platform. The ultimate goal is to allow the end user to retain total control over their account, whilst revealing minimal amounts of metadata to our servers or anyone who might be trying to snoop on our network traffic.

The techniques used by the Feirm Platform have been inspired and used by trusted services like Open Whisper System's Signal Protocol, Blockchain.info's encrypted web-wallet and Off-the-Record Messaging protocol. Each of these systems are still in widespread use today and have undergone heavy security audits from professional cryptographers.

As the primary cryptography mechanisms used by the Feirm Platform to manage keys are executed client-side in the users web browser, there is a possibility of an attacker wanting to inject malicious code into the website. There are two methods of how this could be executed:

* A man-in-the-middle (MITM) attack against between the CDN delivering the Platform web application, and the end user. This is highly unlikely due to SSL encrypting data in transport.
* An attack against any third party JavaScript libraries being used by the Feirm Platform web application. Again, this is unlikely as the web application is not dependent on many third-party libraries at all.

In order to try and limit the attack surface though, we chose not to use any analytics scripts or platforms, such as Google Analytics, as it cannot be guaranteed that the tracking scripts they provide do not contain malicious code. Not to mention that tracking our users is something we are against too. 

## User Accounts
When the user logs in to their Feirm Platform account, it appears they are doing so with their username, password and PIN. This may be the case, but involved is heavy client-side cryptography. For example, we do not authenticate a user with their password, but a unique cryptographic keypair instead.

**Root Key**

The root key is a 32-byte hexadecimal string used as the root of a Feirm Platform account. It can be used to derive identity keypairs and other encryption keys for a user account. It is vital to keep this secure because once it is compromised, the entire account is unsafe.

**Secret Key**

The secret key is initially derived from a user password in order to create a much stronger password. User provided passwords are typically short, predictable and insecure, so we apply cryptography to make it stronger. The Platform web application currently uses the Argon2 key derivation function to achieve this in the browser.

**Identity Keypair**

Each account on the Feirm Platform has a permanent identity keypair associated to it. As the name states, it is a users identity, meaning it is used anywhere from authentication (proving the user owns their identity keypair) all the way through to encryption.

A users identity keypair cannot change throughout the lifetime of their account. On the centralised layer, a users identity public key is stored, meaning there is a possibility of an attack which could alter the public key in the event of a breach. We do plan to mitigate this threat in the future by implementing a form of decentralised identity solution.

### Creating an Account
When the user wants to create a new account, they first need to generate an identity keypair which we just discussed. In order to prove ownership of this identity keypair, the API requires the user to sign a unique message whilst signing up. This process is done in the background however, so no additional actions are required from the end user. Here is the process in a bit more detail.

#### Generating a Root Key
We first must generate a Root Key, or what we call the `AccountKeyRoot`. This is a random 32-byte hexadecimal string derived client-side using the Web Crypto API. It is the base of our account. Here is an example of what a Root Key looks like.

```
781eda7b532c186e9d8c820650c055d9337ed090703f112f5cc3a6c1131f07e5
```

#### Generating an Identity Keypair
Now that we have the Root Key, we can derive 2 more keys to be used throughout the Feirm Platform. It is bad practice to use the same key for everything, hence the reason for this being our Root Key to derive new keys from. These are the 2 keys we will be deriving:

1. Identity Keypair private key (`AccountKeyIdentityPrivate`) - Derived using `SHA256(AccountKeyRoot | "identity")`. Acts as the seed to produce an Ed25519 keypair.
2. Encryption private key (`AccountKeyEnc`) - Used to encrypt other account data (such as contacts) using AES-256. Derived using `SHA256(AccountKeyRoot | "enc")`.

#### Proving ownership of the Identity Keypair
To verify that the user owns their Ed25519 Identity keypair, they are required to fetch an ephemeral token from the API, and sign the nonce. Each ephemeral token has a short life-span of 5 minutes and a unique identifier associated to it. This is primarily used by the API to query the Redis cache and associate tokens between users. Here is an example of an ephemeral token returned by the API.

```json
{
    "id":"caef00e7-ec71-4aea-b338-81a31909b767",
    "nonce":"Sign this message to create your Feirm account. Random nonce: 3850b0dbb0dd177c5e80f383c58e1c9985080eab9ac5b956821bbccd960fb2fe"
}
```

Once the nonce has been successfully signed by the Identity Keypair, we can then encrypt all the sensitive data of a user account using the Secret Key (derived from a users password) and AES-256-CBC.

#### Generating the Secret Key
First of all, we generate a random 16-bytes of Salt (`PassphraseSalt`) to be used for Argon2 key derivation. When it comes to using Argon2, there are a few different versions. For the Feirm Platform, we use the `Argon2id` version which is a hybrid of the other versions and generally the most recommended to use. We also use a hash length of 32 bytes. More information about Argon2 can be found [here](https://en.wikipedia.org/wiki/Argon2).

Now that we have our 16-bytes of Salt, we can generate our Argon2 hash with a length of 32 bytes. Here is an example of the password stretching process:

```
Plaintext Password: password123
PassphraseSalt: 6fcd4bd1c80185e0d14c051f1cc72444

Argon Output
============
Hex: e7f665b8f4f3765a5cbd75726202a90abb0167ad96ea3ba618ee492f75dddff9
Encoded: $argon2id$v=19$m=16,t=2,p=1$NmZjZDRiZDFjODAxODVlMGQxNGMwNTFmMWNjNzI0NDQ$5/ZluPTzdlpcvXVyYgKpCrsBZ62W6jumGO5JL3Xd3/k
```

It is important to note that the API does not store the stretched version of the Secret Key, only the Salt generated so that the Secret Key can be reconstructed at a later date (when signing in from a new device). The point now though is that we have a much stronger key which can be used to encrypt our Feirm Platform account root key.

#### Encrypting the Root Key
Now that we have a strong Secret Key, this can be used to encrypt the Root Key of a user account. Encryption is carried out using AES-256-CBC. In the future we may choose to use a different encryption algorithm, but AES is the gold-standard at the moment.

To get started, we generate some more random salt to be used as our Initialisation Vector (IV) (or `SecretIv`) - again, this is 16-bytes.

We provide the Secret Key, IV and Root Key as inputs to the AES Cipher, and as a result, we are left with the Ciphertext of our encrypted Root Key.

```
Ciphertext: a8626916a69905facf6000e004ae99af19fcf3398e1f6d5099d2425e8696916f
```

#### Sending the data to the API
We now have all the data necessary to construct a payload to be sent to the API. Here is a table containing the data we have generated/derived so far, and whether or not it will be sent to the API:

| Property                  | Submitted to API |
| ------------------------- | ---------------- |
| Username                  | Yes              |
| PIN                       | Yes              |
| Passphrase                | No               |
| SecretKey                 | No               |
| AccountKeyRoot            | No               |
| AccountKeyIdentityPrivate | No               |
| AccountKeyEnc             | No               |
| AccountKeyIdentityPublic  | Yes              |
| PassphraseSalt            | Yes              |
| SecretIv                  | Yes              |
| Ciphertext                | Yes              |
| TokenSignature            | Yes              |

### Logging into an Account
When it comes to logging into an account, the user is required to enter the six-digit PIN from when they created their account. Without the correct PIN, a user connect retrieve the encrypted payload for their account. This mechanism was introduced as an attempt to mitigate brute-force attacks on the encrypted payloads. Paired with rate limiting enforced by the authentication API, it makes trying to brute-force PIN numbers an extremely difficult process. Also not to mention that if an attacker was able to guess a PIN correctly, they would need to decrypt the account payload - which is near impossible at the present day.

Although, there is an element of trust needed from the Feirm project as PIN entry is handled by the authentication API. If the Feirm Platform database were to be leaked, an attacker could get their hands on all of the encrypted account payloads. A PIN number will not help in that regard. It's purely to keep snooping individuals from trying to get access to your encrypted account data.

Although a breach is bad, the sensitive data is encrypted rendering it useless to an attacker. Rainbow table attacks are not possible due to the fact that passwords or Secret Keys are not directly stored by the authentication API. The only time an attacker could potentially re-compute a Secret Key is if a user used a weak password when creating their Feirm Platform account.

There is no way to indicate the strength of a password against the authentication API, which is why the web application uses [zxcvbn](https://github.com/dropbox/zxcvbn) to estimate the strength of a user password on the client-side. We enforce a rule that all user accounts must have a "strong" password, which hypothetically mitigates the ability to decrypt account data based on a weak password.

Here is the typical login process:

1. A user enters their username and password, as well as their PIN.
2. The password is ignored for this step, but a payload containing the username and the six-digit PIN is sent to the API asking for the associated encrypted account data.
3. If the PIN is valid, the request will return the encrypted account data such as `CipherText`, `SecretIv` and `PassphraseSalt`.
4. The password is used for this step - it is salted and stretched client-side in order to derive the `SecretKey`. The process is exactly the same as the processes used when creating an account.
5. If the `SecretKey` can be successfully used to decrypt the encrypted account details, then the attempt is considered as successful. If not, then the user will have to repeat the process again.
6. The nonce returned in the request is signed by the `AccountKeyIdentityPrivate` in order to prove ownership of the Identity Keypair. If the signature is valid on the server-side, a JWT session token is issued, and the user is logged in.