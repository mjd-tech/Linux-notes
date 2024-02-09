# ssh
ssh (secure shell). Server daemon - sshd

- get an interactive terminal session on remote host
- execute commands on remote host
- transfer files
- tunnel TCP traffic

## Examples
```
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

## Important files

### ~/.ssh/config

Client-side config file. stores ssh options for frequently used hosts.
A minimal entry looks like this

    # My server
    Host        hobbiton
    HostName    192.168.1.244
    User        frodo

- "Host" is an alias of the actual hostname
- "HostName" is the actual hostname, it can be an IP address or a DNS
  address
- "User" is the user name on the remote host
- Use a blank line to separate "Hosts"

Other commonly used options:

    Port 2222             # the default port is 22
    ForwardX11 yes        # if remote has gui, you can run gui app via ssh. It will be slow.
    CheckHostIP no        # speeds up slow logins.

#### Global Settings

Put at the top of config file, before host definitions

    # Prevent ssh sessions timing out
    ServerAliveInterval 60
    # Remove delay in initial connection
    CheckHostIP no
    # Human readable known_hosts file
    HashKnownHosts no

#### Jump Hosts

    You ---> Internet ---> Firewall ---> Jump Host ---> Desired Host

You can get to the Jump Host because the firewall allows it. But you
cannot get to Desired Host, the firewall blocks this.

First set up the config information for the jumphost. Ideally, you want
to use shared key (passwordless) authentication.

    Host      jump1
    HostName  jumphost1.example.org
    Port      2222
    User      jumpuser

Next, set up the desired host.
OpenSSH 7.3, released August 2016, makes this easy with the ProxyJump directive.

    Host      desired
    HostName  192.168.1.38
    ProxyJump jump1
    User      fred

Between OpenSSH 5.4 and 7.2 inclusive, a 'netcat mode' can be used.

    Host      desired
    Hostname  192.168.1.38
    ProxyCommand  ssh -q -W %h:%p jump1
    User      fred

The old way prior to OpenSSH 5.4 used netcat, nc(1).

    Host      desired
    Hostname  192.168.1.38
    Port      22
    ProxyCommand  ssh -q jump1 nc -q0 %h %p
    User      fred

### ~/.ssh/known_hosts

Client-side file. Stores the server's public key.

The first time you connect to a server via ssh, you get this message.

    RSA key fingerprint is blah blah.
    Are you sure you want to continue connecting (yes/no)?

Type "yes" and the server's public key gets stored in your
~/.ssh/known_hosts file.

Sometimes when connecting to a computer with SSH, things can get jumbled
up and an error can occur that looks like this:

    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @     WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    ...
    blah blah blah
    ...
    Offending key in /home/user/.ssh/known_hosts:3
    RSA host key for remotehost.mydomain.com has changed and you have requested strict checking.
    ...

This tells you the problem is with **line 3** in known_hosts and it
refers to **remotehost.mydomain.com**.

3 ways to fix it:

    ssh-keygen -R remotehost.mydomain.com
    vi ~/.ssh/known_hosts  # use your favorite editor
    sed -i".bak" '3d' ~/.ssh/known_hosts

Make an ssh connection without the host being added to your known_hosts file.

    ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null user@host

For repeated use, consider adding to your ~/.ssh/config

    Host somehost.somedomain 
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
    User foo
    LogLevel QUIET

### ~/.ssh/authorized_keys

Server-side file. Lets the server authenticate the user. Holds public
keys of remote users that are allowed to log into the local user’s
account without providing a password.

## SSH - No Password

It's recommended to add the remote host to your ~/.ssh/config file.

Step 1: Create public and private keys on local host

    # Just hit Enter to respond to all questions.
    ssh-keygen

Step 2: Copy your public key to remote-host using ssh-copy-id

    # When prompted, enter the password for the user on the remote host 
    ssh-copy-id -i my-remote-host

Step 3: See if it works

    ssh my-remote-host

## XWindows over SSH

Make sure the following are set on the server side:

    X11Forwarding yes
    X11UseLocalhost no

- start the client with ssh -Y or ssh -X
- run the desired app from the command line.
- You can run an entire desktop session. Not recommended, poor performance.

## Port Forwarding (Tunnelling)

Example: Connect to MySQL via ssh tunnel

    ssh -fN -L 3306:localhost:3306 user@ssh_gateway
    user@ssh_gateway's password: ******

    mysql -u root -h 127.0.0.1 -P 3306
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    ...
    mysql> exit
    Bye

Note: -f puts ssh into background, -N tells ssh you are not sending any
commands, just set up tunnel, -L sets up the actual tunnel.  
The *localhost* in 3306:localhost:3306 is from the perspective of the
ssh_gateway, i.e. itself.

If running MySQL locally on port 3306, use a different port for the tunnel:

    $ ssh -fN -L 33306:localhost:3306 user@ssh_gateway
    user@ssh_gateway's password: ******

    $ mysql -u root -h 127.0.0.1 -P 33306
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    ...
    mysql> exit
    Bye

SSH treats *localhost* and *127.0.0.1* the same. They are synonymous.  

MySQL treats *localhost* and *127.0.0.1* differently.  

`mysql -h localhost` uses a unix socket to connect,  
`mysql -h 127.0.0.1` uses TCP/IP.

If MySQL is on a different host than ssh_gateway

    $ ssh -fN -L 33306:remote_db_server:3306 user@ssh_gateway

remote_db_server must be resolvable from the ssh_gateway's perspective.

### Remove ssh tunnel
A tunnel created with -f (background) will remain until you manually kill the ssh process
that launched the tunnel. 

Use this trick to create a temporary tunnel, that goes away when you are through using it.

    ssh -f -L 3306:localhost:3306 ssh_gateway sleep 30
    mysql -u root -h 127.0.0.1 -P 3306

You have 30 seconds to launch the mysql command. Otherwise the tunnel goes away.
Once you launch mysql, the tunnel remains until you exit mysql.

## Remote Commands
When running these, ssh exits with the status of the last remote
command, or with status 255 if ssh could not execute the remote command.

**Single command**

    ssh user@remotehost 'ls'

**Multiple commands**

    ssh user@remotehost 'pwd; ls'

    -or-

    ssh user@remotehost "pwd && ls"

**Multiple commands, heredoc style**

    ssh user@remotehost /bin/bash <<'ENDSSH'
    pwd
    ls
    ENDSSH

Note that executing the above without /bin/bash will result in the
warning *Pseudo-terminal will not be allocated because stdin is not a
terminal*. Also note that ENDSSH is surrounded by single-quotes, so that
bash recognizes the heredoc as a *nowdoc*, turning off local variable
interpolation so that the command text will be passed as-is to ssh.

**Run command with sudo**

    ssh -t user@remotehost 'pwd; sudo ls'

For more complex operations it is easiest to write a script file
locally, then execute it on remote host.

**Execute local script on remote host**

    ssh user@remotehost 'bash -s' < localscript
    -or-
    cat localscript  | ssh user@remotehost
    -or-
    ssh user@remote_server "$(< localscript)"

The last one allows interactive commands e.g. sudo

If the local script accepts arguments:

    ssh user@remotehost 'bash -s' -- < localscript arg1 arg2 --argwithdash(es)

The double dash after 'bash-s' tells bash there are no more arguments -
to bash!

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
```
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
```
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

### Problem 3 old ssh servers with unsupported key exchange and cipher types

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

