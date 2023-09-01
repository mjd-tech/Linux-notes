# SSL and TLS Overview

- Protocol for encrypting communications over a computer network.
- Secure Sockets Layer (SSL) came first, developed for Netscape Browser.
- Transport Layer Security (TLS) came later, evolved from SSL, but both
  are frequently referred to as SSL
- Common uses: web browsing, email, file transfer, instant messaging,
  and voice-over-IP (VoIP).

## Symmetric Encryption

- Encryption and Decryption use the same key.
- Simple example: "ROT-13", which shifts the letters in the alphabet 13
  places.
- More effective is a randomly generated key.
- The problem is getting the same key to the sender and receiver,
  without sending the raw key over the wire.
- If the key is intercepted, the message can be decrypted by someone
  other than the intended recipient.

## Asymmetric (Public Key) Encryption

- Requires two keys to encrypt/decrypt plain text: Public and Private
  keys, generated as a pair.
- The two entities exchange public keys.
- Each party uses the other’s public key to encrypt the message.
- The other party uses its own private key to decrypt the message.
- Also, a message can be encrypted with the private key and decrypted
  with the public key, this is used with certificates (below).

In Practice:

- Asymmetric Encryption is "expensive" in terms of computing power.
- In general only one message is sent using a public key cryptosystem:
- "Here's a randomly generated key for a symmetric algorithm, let's both
  use that from now on."

## Certificates

Let's say you are doing some online banking. You'll be using https,
which is http over a secure (TLS) channel. You want to make sure you are
talking to the actual bank, not some man-in-the-middle attacker.

This is where the Certificate comes in.

- A Digital Certificate that identifies the Bank's website and contains
  the Bank's Public Key.
- The Certificate is issued by a 3rd party Certificate Authority (CA),
  encrypted with the CA's private key.

How it works.

- Client browser connects to bank website (server).
- Client and server negotiate which version of TLS and which encryption
  algorithms to use (handshake).
- Server sends its certificate. This cert is encrypted with the CA's
  private key.
- Client verifies the certificate using the known CA Public keys that
  came with the browser.
- Client browser generates a one-time key for the session, encrypts it
  with the server's public key (it's part of the certificate), and sends
  it to the server.
- The server decrypts the one-time key, using its own private key,
- then uses that key together with the agreed symmetric algorithm for
  the rest of the session.
