# Cryptography Layer

## Introduction
As of 2018, modern web browsers have included the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) which allows for using JavaScript to perform cryptographic operations such as encryption, hashing and verifying digital signatures.

The Feirm Marketplace has made use of these cryptographic operations in order to secure information and enable end-to-end secret messaging between users on the platform. The ultimate goal is to allow the end user to retain total control over their account, whilst revealing minimal amounts of metadata to the Feirm Marketplace servers or anyone who might be snooping.

The techniques used by the Feirm Marketplace have been inspired and used by trusted services like Open Whisper System's Signal Protocol, Blockchain.info's encrypted web-wallet and Off-the-Record Messaging protocol. Each of these systems are still in widespread use today and have undergone heavy security audits from professional cryptographers.

As the primary cryptography mechanisms used by the Feirm Marketplace to manage keys are executed client-side in the users web browser, there is a possibility of an attacker wanting to inject malicious code into the website. There are two methods of how this could be executed:

* A man-in-the-middle (MITM) attack against the webserver where the Feirm Marketplace is hosted.
* An attack against any third party JavaScript libraries used by the Feirm Marketplace website.

In order to limit this attack surface, we chose not to use any third-party analytics scripts such as Google Analytics as it cannot be guaranteed that they will always be secure. Not to mention the invasive data collection involved too.

In the future, once the Feirm Marketplace client is further developed, we will make it open source so that users can run the website locally instead of having to rely on our webserver.

## User Accounts
When the user logs in to their Feirm Marketplace account, it appears they are doing so with their username and password. This may be the case, but involved is heavy client-side cryptography that is ongoing in the background. For example, the password is a type of secret key which never leaves the users web-browser.

**Secret Key**

The secret key is determined by transforming the users password into a secure key using a key stretching mechanism like Argon2. A further key is encrypted with the stretched key and this is used for actions on the Feirm Marketplace.

**Public Identity Key**

Each account on the Feirm Marketplace has a permanent public identity key pair. This is used to authenticate the ownership of the account as well as various other actions on the Marketplace.

On a Feirm Marketplace users profile, the public key of their account is shown. Other users who are visiting or interacting with this user are encouraged to make a note of this public key to ensure that it does not change. Whilst the user cannot change their public key, there is a possibility of a man-in-the-middle or hijacking attack that could alter the public key. The Feirm Marketplace does plan to combat these attacks by relying on an external PKI or DPKI.

### Creating an Account
When the user wants to create a new account, they first need to generate a public and private key pair. In order to prove ownership of the public key later on, the API requires the user to sign a unique message whilst signing up.

Upon signing up, a request is made to fetch a generate a new ephemeral registration "token" from the API. It contains two parts:

* `Id` - A unique identifier
* `Nonce` - A unique message to sign with the private key

Now that the token has been retrieved from the API, the user needs to generate an account keypair. The next step is to generate a 32-byte seed which is used as the base for all cryptographic operations to take place throughout an account - **it is vital to keep this secure**. We will call this the `AccountKeyRoot`.

Once we have our `AccountKeyRoot`, we need to derive two more keys from it:

1. `AccountKeyIdentityPrivate` - this will be our private key used for signing messages using the secp256k1 elliptic curve. It is derived using `SHA256(AccountKeyRoot + "identity")`
2. `AccountKeyEnc` - this is used for AES-256 encryption. It is derived using `SHA256(AccountKeyRoot + "enc")`

The unencrypted `AccountKeyRoot` **never** leaves the users device. It always stays in the browser. For this reason, to ensure a user cannot be locked out of their account just by forgetting their `AccountKeyRoot`, we store an encrypted version on the Feirm Marketplace server. Instead of a password, we use a stretched version of the users password so that it is more secure and makes brute-force attacks near-impossible. This is our secret key that we discussed earlier.

In order to transform a user's plaintext password into our secret key, we use the key derivation function [Argon2](https://en.wikipedia.org/wiki/Argon2). The first implementation of the cryptography for the Feirm Marketplace evolved around PBKDF2, but Argon2 allows for better password-cracking resistance. Transforming a password into a more stronger key is known as *"key-stretching"* and adds additional computational work to make it extremely difficult to crack a password. The user's password is taken and stretched using Argon2 with salt `PassphraseSalt` to create our secret key `SecretKey`. Here is a breakdown of these values:

* `PassphraseSalt` - 16 random bytes used for Argon2 salt

Now that we have our `SecretKey`, we can use it to encrypt our `AccountKeyRoot` using AES-256-CBC. We generate an additional 16 random bytes to act as our initialisation vector `SecretIv`. The ciphertext of the encryption will be known as `CipherText`

Lastly, the final thing we have to do is sign the `Nonce` of our token we received at the beginning. To do this, we first generate a Keccak-256 hash of the `Nonce`. Our signature (`TokenSignature`) is an ECDSA secp256k1 signature of the Keccak-256 hash of `Nonce`.

Once all of this information has been generated, the non-sensitive data is submitted to the Feirm Marketplace API. Below is a table of all the data generated and submitted:

| Property                  | Submitted to API |
| ------------------------- | ---------------- |
| Passphrase                | No               |
| SecretKey                 | No               |
| AccountKeyRoot            | No               |
| AccountKeyIdentityPrivate | No               |
| AccountKeyEnc             | No               |
| AccountKeyIdentityPublic  | Yes              |
| PassphraseSalt            | Yes              |
| SecretIv                  | Yes              |
| CipherText                | Yes              |
| TokenSignature            | Yes              |
| Username                  | Yes              |
| TOTPSecret                | Yes              |

### Logging into an Account
When it comes to logging into an account, mandatory two-factor authentication is enforced. When the user first created an account, they would have been prompted to configure 2FA in order to finalise the account creation process. In the future, we hope to implement other two-factor authentication such as U2F (for hardware keys). 

When it came to revising the technical specifications for the Feirm Marketplace, it was decided not to require an email address to use the marketplace. Originally, the users email address was used for mandatory two factor authentication where a confirmation link would be sent to their inbox in order to approve the login. However, whilst this method increases the security of an account compared to no two-factor authentication at all, it still isn't foolproof for the following reasons:

* Assuming that an attacker already has knowledge of your Feirm Marketplace password, then the chances are that they have already gained access to your email account, meaning that the attacker can approve the login request themselves.
* The increase of phishing emails has made it easy for attackers to gain access to accounts by taking the user to a fake web-page and asking them to enter their account credentials. If there is no email associated to a Feirm Marketplace account, then the user is most likely to note that this is a phishing attempt if they ever receive an email claiming to be from us.
* Emails are one more piece of identifiable data. Yes, anonymous email providers exist, but using the same email for multiple services forms a digital trail of your activities online. Not to mention that if there was a data breach on the centralised layer, this information could be sold to others against your will.

Two-factor authentication has always been made mandatory in order to protect the encrypted `AccountKeyRoot` which comprises of `CipherText` and `SecretIv`. Even though the process is slowed due to key-stretching, passwords are still theoretically susceptible to brute-force attacks. Hiding the encrypted key behind a TOTP two-factor authentication method eliminates this hypothetical attack as the encrypted information will not be returned in the request unless the temporary passcode is valid.

Although, there is an element of trust needed from us as the two-factor authentication method is handled by the API. If the Feirm Marketplace database were to be leaked, an attacker could get all of the encrypted private keys. Although a breach is bad, the confidential data is encrypted rendering it useless to an attacker. Rainbow table attacks are not possible due to passwords being salted and stretched, and any other type of brute-force attacks are extremely hard. The only time the hacker could obtain a password is if it was very weak in the first place (most likely a commonly used password) or if the password was re-used from another compromised database.

Here is the typical login process:

1. A user enters their username and password, as well as a time-based passcode (TOTP).
2. The password is ignored for this step, but a payload containing the username and the time-based passcode is sent to the API asking for the associated encrypted private key.
3. If the time-based passcode is valid, the request will return the encrypted account details such as `CipherText`, `SecretIv` and `PassphraseSalt`.
4. The password is used for this step - it is salted and stretched client-side in order to derive the `SecretKey`. The process is exactly the same as the processes used when creating an account.
5. If the `SecretKey` can be successfully used to decrypt the encrypted account details, then the attempt is considered as successful. If not, then the user will have to repeat the process again.
6. The nonce returned in the request is signed by the `AccountKeyIdentityPrivate` in order to prove ownership of the keypair. If the signature is valid on the server-side, a JWT is issued containing the `AccountId`, `Iat` and `Exp`.