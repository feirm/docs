# Centralised Layer

## API
The Feirm Committee maintain the API which provides functionality to the Feirm Marketplace. The API is operated from a secure data-centre located in Germany.

The Feirm Marketplace API is not documented due to constantly being under development. There are many features constantly being integrated. However, once this slows down and we reach more stable releases, documentation on the endpoints will be released.

## Accounts
When first using the Feirm Marketplace, you need to create an account. Your account is known by a public username that cannot be changed.

Instead of a password used to protect data on the server-side, all data is encrypted client-side before being uploaded through the API. Additionally, to further secure a user account, we have made TOTP two-factor authentication mandatory. An application like Google Authenticator or Authy can be used to generate these time-sensitive secrets to assist with logging into an account.

## Marketplace Moderation
In order to ensure that the Feirm Marketplace is safe from abuse, we have a responsibility to remove certain users and content from the platform. A user can be suspended if they are caught doing the following actions:

* Harassing other users (through spam etc)
* Attempting to defraud other users
* Selling illicit items such as drugs

The moderation team compromises primarily of Feirm Committee members, but we hope to branch out and involve trusted community members once the Marketplace grows in size. Although the moderation team aims to handle all forms of abuse, we cannot catch everyone - especially when communicating with other users using the end-to-end encrypted chat.

In the event where abuse is present in an encrypted chat, the user can volunteer their shared key in order to decrypt the encrypted messages. If a user is also purposefully reporting innocent users, this action is considered as spam and will result in their user account being suspended.

## Encryption
The API stores and transports encrypted data between users. This data is not decipherable without having the necessary keys, which are stored on the user's device. Since private data on the Feirm Marketplace is end-to-end encrypted, the Feirm Marketplace staff or any intruder cannot decipher the information hosted on our servers.