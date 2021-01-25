# Centralised Layer

## API
The Feirm project maintain the APIs which provides functionality to the Feirm Platform. The API is operated from a secure data-centre located in Germany.

The Feirm Platform API is not documented due to constantly being under development. There are many features constantly being integrated. However, once this slows down and we reach more stable releases, documentation on the endpoints will be released.

## Accounts
When first using the Feirm Platform, you need to create an account. Your account is known by a public username that cannot be changed.

Instead of a password used to protect data on the server-side, all data is encrypted client-side before being uploaded through the API. Additionally, to further secure a user account, we have enforced a six-digit PIN to protect your account data.

## Encryption
The API stores and transports encrypted data between users. This data is not decipherable without having the necessary keys, which are stored on the user's device. Since private data on the Feirm Platform is end-to-end encrypted, the Feirm project or any intruder cannot decipher the information hosted on our servers.