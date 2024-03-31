# ssh
ssh (secure shell). Server daemon - sshd

- get an interactive terminal session on remote host
- execute commands on remote host
- transfer files
- tunnel TCP traffic

## Examples
```bash
# Connect to host on the default port 22
ssh user@host.example.com

# Specify port
ssh -p 2222 user@host.example.com

# Use host definition in ~/.ssh/config
ssh myhost

# Run a command on remote host
ssh myhost cat /etc/hosts

# Multiple commands
ssh myhost 'pwd; ls'

# Multiple commands, heredoc style
ssh myhost /bin/bash <<'ENDSSH'
pwd
ls
ENDSSH

# Note: executing the above without /bin/bash results in warning
# "Pseudo-terminal will not be allocated because stdin is not a terminal".
# 'ENDSSH' is surrounded by single-quotes, turns off variable interpolation
# so that the command text will be passed as-is to ssh.

# Run command with sudo
ssh -t myhost 'pwd; sudo ls'

# Execute local script on remote host (3 methods)
ssh myhost 'bash -s' < localscript
cat localscript | ssh myhost   # another way
ssh myhost "$(< localscript)"  # allows interactive commands e.g. sudo

# If the local script accepts arguments:
ssh myhost 'bash -s' -- < localscript arg1 arg2 --argwithdash(es)

# Note: double dash after 'bash -s' means there are no more arguments to bash

# Create an ssh tunnel for MySQL 
ssh -fN -L 3306:localhost:3306 remotehost

# Note: -f puts ssh into background, -N tells ssh to just set up tunnel,
# -L sets up the actual tunnel.
# "localhost" is from the perspective of "remotehost" not your local host
```

## SSH - No Password

It's recommended to add the remote host to your ~/.ssh/config file.

### Step 1: Create public and private keys on local host

```bash
# Just hit Enter to respond to all questions.
ssh-keygen
```
This generates:
- `~/.ssh/id_rsa` (private key)
- `~/.ssh/id_rsa.pub` (public key)

By default, keys have a "comment" initialized to "user@host"

To change the comment:
```bash
ssh-keygen -f ~/.ssh/id_rsa -c
```

> 📝 Note:  
> Starting with OpenSSH 9.5/9.5p1 (2023-10-04):
> - ssh-keygen generates Ed25519 keys by default.
> - Ed25519 public keys are smaller and more secure than rsa keys
> - Ed25519 keys are supported since OpenSSH version 6.5 (January 2014)

### Step 2: Copy public key to remote-host using ssh-copy-id
```bash
# When prompted, enter the password for the user on the remote host 
ssh-copy-id -i my-remote-host
```

### Step 3: See if it works

```bash
ssh my-remote-host
```

### SSH for Github
Github requires either ed22519 or rsa 4096
```bash
ssh-keygen -t ed25519 -f github -C "your_email@example.com"
# or
ssh-keygen -t rsa -f github -b 4096 -C "your_email@example.com"
```
Put this in `~/.ssh/config`
```
# Github
Host        github.com
HostName    github.com
IdentityFile ~/.ssh/github
```

## XWindows over SSH

Make sure the following are set on the server side:

    X11Forwarding yes
    X11UseLocalhost no

- start the client with ssh -Y or ssh -X
- run the desired app from the command line.
- You can run an entire desktop session. Not recommended, poor performance.

## Port Forwarding (Tunnelling)

Example: Connect to MySQL via ssh tunnel
```bash
ssh -fN -L 3306:localhost:3306 user@ssh_gateway
user@ssh_gateway's password: ******

mysql -u root -h 127.0.0.1 -P 3306
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
mysql> exit
Bye
```

Note: -f puts ssh into background, -N tells ssh you are not sending any
commands, just set up tunnel, -L sets up the actual tunnel.  
The *localhost* in 3306:localhost:3306 is from the perspective of the
ssh_gateway, i.e. itself.

If running MySQL locally on port 3306, use a different port for the tunnel:
```bash
ssh -fN -L 33306:localhost:3306 user@ssh_gateway
user@ssh_gateway's password: ******

mysql -u root -h 127.0.0.1 -P 33306
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
mysql> exit
Bye
```

SSH treats _localhost_ and _127.0.0.1_ the same. They are synonymous.  

MySQL treats _localhost_ and _127.0.0.1_ differently.  

`mysql -h localhost` uses a unix socket to connect,  
`mysql -h 127.0.0.1` uses TCP/IP.

If MySQL is on a different host than ssh_gateway

    $ ssh -fN -L 33306:remote_db_server:3306 user@ssh_gateway

remote_db_server must be resolvable from the ssh_gateway's perspective.

### Remove ssh tunnel
A tunnel created with -f (background) will remain until you manually kill the ssh process that launched the tunnel. 

Use this trick to create a temporary tunnel, that goes away when you are through using it.

```bash
ssh -f -L 3306:localhost:3306 ssh_gateway sleep 30
mysql -u root -h 127.0.0.1 -P 3306
```

You have 30 seconds to launch the mysql command. Otherwise the tunnel goes away.
Once you launch mysql, the tunnel remains until you exit mysql.

## SSHd Server
The config file is `/etc/ssh/sshd_config`  
Here you can set the TCP port (default 22) sshd listens on.

A couple of other things to look at:

**PermitRootLogin**: Specifies whether root can log in using ssh.
- yes - permit root to login with password or with ssh keys
- no - root cannot use ssh at all
- prohibit-password - (default) root must use ssh key.
- forced-commands-only - root allowed (ssh key only), but only to run a
  command (no interactive session)

## Annoyances
SSH uses "key exchange algorithms", "ciphers" and "key types".
Over time, the old ones get deprecated because they're not secure enough.

### Problem 1: Old ssh client (such as Toad for Windows) - new ssh server.

The issue is with the key exchange algorithm and the cipher.

**Solution:**

on the SERVER:
```bash
cd /etc/ssh/sshd_config.d
sudo vi legacy.conf
```
Put this in:
```
# Legacy changes
KexAlgorithms +diffie-hellman-group1-sha1
Ciphers +aes128-cbc
```
Then restart the ssh server
```bash
sudo systemctl restart ssh.service
```

### Problem 2: New ssh client - old ssh server.

- The issue is with public key sent by the old server.
- It sends a ssh-rsa key encoded with SHA-1 algorithm.
- SHA-1 is deprecated as of openssh 8.8. You're supposed to use SHA-2
- But you can't tell if a public key is SHA-1 or SHA-2 by looking at it.

**Solution:**

- on the CLIENT, in ~/.ssh/config
- Go to the host definition that's causing problems, add this:

```
PubkeyAcceptedKeyTypes +ssh-rsa
```

If no host definition in `~/.ssh/config`, modify ssh command line like this:

```
ssh -o PubkeyAcceptedKeyTypes=ssh-rsa ...blah-blah...
```

**Note:**

- This gets confusing.
- The name "ssh-rsa" has **two different meanings**.
- There's the _key type_ ssh-rsa, in the first column of authorized_keys file.
- RSA keys are perfectly fine and widely supported.
- However, there's also the _signature algorithm_ ssh-rsa which uses the SHA-1 hash algorithm.
- SHA-1 is insecure, OpenSSH disables ssh-rsa _signature algorithm_ since version 8.8.
- SSH clients and servers are now expected to use rsa-sha2-256 or rsa-sha2-512

### Problem 3: old ssh servers with unsupported key exchange and cipher types

you may encounter the following errors:

    ssh host2.example.com
    Unable to negotiate with 192.168.0.2 port 22: no matching key exchange method found.
    Their offer: diffie-hellman-group1-sha1

    ssh -oKexAlgorithms=diffie-hellman-group1-sha1 host2.example.com
    Unable to negotiate with 192.168.0.2 port 22: no matching cipher found.
    Their offer: aes128-cbc,3des-cbc,aes192-cbc,aes256-cbc

Temporary Fix:

    ssh -oKexAlgorithms=diffie-hellman-group1-sha1 -c aes256-cbc host2.example.com

Permanent Fix

add the following lines to `~/.ssh/config`

    # Single Host
    Host host2.example.com
    KexAlgorithms diffie-hellman-group1-sha1
    Ciphers aes256-cbc

    # Multiple Hosts
    Host *.example.com
    KexAlgorithms diffie-hellman-group1-sha1
    Ciphers aes256-cbc

    # Multiple KexAlgorithms and Cipers, separted by commas, can be specified as needed
    Host *.example.org
    KexAlgorithms diffie-hellman-group1-sha1,diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1
    Ciphers aes128-cbc,3des-cbc,aes192-cbc,aes256-cbc

### Get public keys from remote ssh server:
```
ssh-keyscan user@remotehost
```