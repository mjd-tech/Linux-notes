# Encryption Notes

**Most common uses:**

- Data in transit - HTTPS, SSH, VPN etc
- Data at rest - Files on stored on server or computer drive.

**Keys:**

- You "lock" (encrypt) your data.
- You "unlock" (decrypt) your data.

**Key pairs:**

- Private key aka secret key
- Public key

**Private key:**

- Can encrypt data
- Can decrypt data
- You can derive a public key from a private key
- Never distribute to other people

**Public key:**

- Can only encrypt data.
- Cannot decrypt data.
- Cannot derive the private key.
- Can distribute to other people.

**Symmetric (Private Key) Encryption:**

- aka "shared key" encryption
- Private key is used for both encryption and decryption.
- Public key not used.
- Faster than asymmetric

**Asymmetric (Public Key) Encryption:**

- Public key is used to encrypt the data
- Private key is used to decrypt the data
- Slower than symmetric. More complicated process.

**Hybrid Encryption:**

- uses asymmetric for "key exchange"
- then switches to symmetric (because of better performance)

## HTTPS

Uses Hybrid encryption, with Authentication.

**Authentication:**

- web browser connects to server, unecrypted.
- Server sends Certificate (a public key) to browser.
- Browser verifies certificate against a list of Certificate
  Authorities.
- Browser complains if server has "self-signed certificate" (not in the
  list of Certificate Authorities)
- You can manually add a "self-signed" certificate to the browser so it
  will stop complaining.

Once the browser is satisfied with the server's certificate, the "key
exchange" process begins.

- Browser sends some random data to the server, encrypted with the
  server's public key. (Asymmetric encryption)
- Both sides **independently** generate the **same** private key from
  this random data.
- From now on, server and client use this private key (Symmetric
  encryption)
- The key is discarded when the session ends.
- It's extremely difficult/expensive for a 3rd party to derive this key.

## SSH

Roughly the same as HTTPS except no Certificates.

Instead of Certificates, you "authenticate" by either

- password
- SSH keys

**To use SSH keys:**

- copy your public SSH key to the server. ( command: ssh-copy-id
  user@server )
- From now on you don't need to enter a password.
- Your public key (a file on the server), and your private key (a file
  in your .ssh directory) go together.

Keep in mind these SSH keys are only used for authentication. They are
not used to encrypt/decrypt the data over the wire.

## PGP - OpenPGP - GPG

These are effectively the same thing.

- PGP (Pretty Good Privacy) - proprietary
- OpenPGP - Open Source
- GPG (Gnu Privacy Guard) - GNU's implementation of OpenPGP
- GPG is standard on Debian/Ubuntu
- The GUI app is "Passwords and Keys"
- The command line app is gpg

Note, the GUI has a bug, it does not import keys.

PGP/GPG is primarily used to encrypt static files and email.

- When you create a gpg key, it asks for your email address.
- If you intend to use this key for email encryption, put your real
  email address.
- Otherwise, can use a dummy address like foo@example.com or even omit
  altogether.
- Also, it asks for a passphrase.
- This is to protect your private key.
- If a 3rd party obtains your private key, they cannot decrypt anything
  without knowing the passphrase.

Email encryption is Asymmetric:

- You send your public key (with your correct email address) to another
  person
- They use the key to encrypt email that they send to you.
- You decrypt with your private key.
- This is handled automatically by the email client (Thunderbird,
  Outlook, etc)
- The client stores the passphrase so you don't need to enter it each
  time you read encrypted email.

File encryption can be either symmetric or asymmetric.

**Armor:**

- by default gpg encryption produces a binary blob.
- email systems work with Text characters only.
- armor converts the binary blob to a text blob.
- The text blob has a header ----BEGIN PGP MESSAGE-----
- and a footer ----END PGP MESSAGE-----
- Armor provides no additional security.
- It's just a way to transport data over systems that expect text.

### Export gpg keys to another machine

Three ways:

- Easiest way: just copy the **.gnupg** directory. You're done.
- Manually export a key to a file. Then import on the other machine
- Export/import via ssh, without creating a file

#### Manual export to file and manual import

    gpg --export-secret-key SOMEKEYID > somekey.sec

Take this file to the other machine, and run:

    gpg --import < somekey.sec

What to use for SOMEKEYID? First run:

    gpg -k

    /home/fred/.gnupg/pubring.kbx
    -----------------------------
    pub   rsa2048 2022-01-13 [SC]
        C6A7DB4FFC7362BED4CCCA14CBC265A8836B4D0B
    uid           [ultimate] Fred Flintstone <fred@bedrock.com>
    sub   rsa2048 2022-01-13 [E]

Option A: (easiest) Use the **uid** as the SOMEKEYID. All of these will
work:

- "Fred Flintstone"
- "fred@bedrock.com"
- "Fred" (provided there are no other keys with "Fred" in the uid)
- just make sure it's unambiguous.

Option B:

- the 40 character hexadecimal blob in the "pub" field.
- the rightmost 16 characters of the 40 character blob

<!-- -->

    # Reduce 40 character blob to 16 character blob
    cut -c 25- <<< C6A7DB4FFC7362BED4CCCA14CBC265A8836B4D0B
    CBC265A8836B4D0B

#### SSH export/import

If you're on the machine that already **has** the key:

    gpg --export-secret-key SOMEKEYID | ssh othermachine gpg --import

If you're on the machine that **needs** the key, you might try:

    # This might not work!
    ssh othermachine gpg --export-secret-key SOMEKEYID | gpg --import

    # So try this:
    ssh othermachine gpg --passphrase-fd 0 --pinentry-mode loopback \
      --export-secret-key SOMEKEYID | gpg --import

If that doesn't work:

- ssh into the machine that has the key.
- Follow above directions for "If you're on the machine that has the
  key"
- Or, do the manual export import
