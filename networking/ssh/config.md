# ssh config file

**Example config file**

```
### Global Settings

# Prevent ssh sessions timing out
ServerAliveInterval 60
# Remove delay in initial connection
CheckHostIP no
# Human readable known_hosts file
HashKnownHosts no

### My Hosts

# NAS, OpenMediaVault, Docker
Host        mynas
HostName    earth
Port        22
User        fred

# RasPi3B+, via port forwarding
Host        raspi
HostName    foobarbaz.duckdns.org
Port        22123
User        pi
ForwardX11  yes

### Remote Hosts

# Linode Server, Debian
Host        mercury
HostName    mercury.example.com
Port        22
User        barney
```

## Host Definintions

```
# Comment describing the host
Host        An alias or nickname
HostName    One of the following:
                A DNS address
                a hostname in /etc/hosts
                an IP address (not recommended, prefer /etc/hosts)
Port        Which TCP port to use, defaults to 22
User        The username on the remote host

```
## Run gui apps over ssh

Add this to host definition.

```
ForwardX11 yes
```

ssh into the host and launch the application. ex. pluma

## automatically run midnight commander when you connect.

Add this to host definition.

```
RequestTTY    yes
RemoteCommand mc
```

## Jump host

- You want to connect to host "foo", but it's firewalled.
- However you CAN get to host "bar", on the same LAN as "foo".
- First, configure host "bar" in your local ~/.ssh/config.
- Then, configure host "foo" like this.

```
# Uses jump host.
Host        foo
HostName    foo.local
Port        22
User        wilma
ProxyJump   bar
```

Note:
- HostName is evaluated from "bar's" perspective. Not yours.
- To make sure it's going to work, manually ssh into bar
- then type: `ssh wilma@foo.local` and make sure it connects.

## Port Forwarding

- You want to connect to MySQL on a remote host "foo".
- MySQL only accepts connections from itself (foo).
- But, you CAN ssh into "foo".

```
# Port forwarding to MySQL
Host        foo-mysql
HostName    foo.example.com
Port        22
User        fred
LocalForward 33306 localhost:3306
ForkAfterAuthentication yes
StdinNull   yes
RemoteCommand sleep 10
```

You can now access MySQL on the remote host like so:
```
ssh foo-mysql && mysql -h 127.0.0.1 -P 33306 -u root -p
```
- Prompts for mysql password, then puts you in mysql shell
- When you quit the mysql shell, the ssh port forwarding terminates
- This is accomplished with the "sleep 10" trick
- StdinNull prevents the terminal from doing strange things

If you want the port forwarding to always stay open:

```
#StdinNull   yes
#RemoteCommand sleep 60
SessionType none
```
Now, the port forwarding will remain active until you kill the ssh
process (use htop, and DON'T kill ssh-agent)
