# Important files

## ~/.ssh/config

Client-side config file. stores ssh options for frequently used hosts.
A minimal entry looks like this

    # My server
    Host        hobbiton
    HostName    192.168.1.244
    User        frodo

- "Host" is an **alias** of the actual hostname
- "HostName" is the actual hostname, it can be an IP address or a DNS address
- "User" is the user name on the *remote** host
- Use a blank line to separate "Hosts"

Other commonly used options:

    Port 2222             # the default port is 22
    ForwardX11 yes        # if remote has gui, you can run gui app via ssh. It will be slow.
    CheckHostIP no        # speeds up slow logins.

### Global Settings

Put at the top of config file, before host definitions

    # Prevent ssh sessions timing out
    ServerAliveInterval 60
    # Remove delay in initial connection
    CheckHostIP no
    # Human readable known_hosts file
    HashKnownHosts no

### Jump Hosts

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

## ~/.ssh/known_hosts

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

## ~/.ssh/authorized_keys

Server-side file. Lets the server authenticate the user. Holds public
keys of remote users that are allowed to log into the local user’s
account without providing a password.

## Permissions
This is the easiest way:
- .ssh directory 700
- all files within 600

