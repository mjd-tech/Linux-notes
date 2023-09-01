# SSH Keys

SSH uses keys in two ways.

1. **Public Key Authentication**: connect without using **passwords**.
2. **Remote Host Verification**: prevents "man in the middle attacks".

Keys are always used in **pairs**, like a safe deposit box

- Private: always stays on the host
- Public: goes to the "other" host

## Public Key Authentication

If you DO NOT already have `id_rsa` and `id_rsa.pub` in your `~/.ssh` directory:

    ssh-keygen -c "SOMETHING"
    # Replace SOMETHING with your name or initials or email address

Copy your public key to the remote host:

    ssh-copy-id -i ~/.ssh/id_rsa.pub remotehost

This appends your public key to the remote host's `authorized_keys` file.

From now on, no more password prompt. The ssh session is authenticated because
your private key (on your host), pairs with your public key (on the remote host)

## Remote Host Verification
When you connect to a remote host, it sends you its public key.

If it's the **first time**
- you are asked to confirm
- the key is is stored in your `~/.ssh/known_hosts` file.

The **next time**
- the remote sends its public key, and it is compared to `~/.ssh/known_hosts`
- If there's a mismatch, ssh displays an error message.

