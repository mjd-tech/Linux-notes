# SSH notes
In case ssh "shared-key" (passwordless) authentication is not configured
and needs to be set up from scratch.

- ensure NAS is in `/etc/hosts`
```
mynas 192.168.1.10
```
test it with `ping -c4 mynas`. should get 4 sucessful pings.

create/edit `~/.ssh/config` file
```
Host        mynas
HostName    mynas
Port        22
User        fred
```
test it with `ssh mynas`. should ask for password, it wants the NAS password

- set up ssh keys, copy to NAS.

```
ssh-keygen -c "SOMETHING"
# Replace SOMETHING with your name or initials or email address

# Copy your public key to the remote host:
ssh-copy-id -i ~/.ssh/id_rsa.pub mynas

# test it
ssh mynas       # should put you into ssh session with no password prompt
```

