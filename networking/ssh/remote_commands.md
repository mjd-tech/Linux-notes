# Remote Commands
When running these, ssh exits with the status of the last remote
command, or with status 255 if ssh could not execute the remote command.

## Single command

```bash
ssh user@remotehost 'ls'
```

## Multiple commands

```bash
ssh user@remotehost 'pwd; ls'
# or
ssh user@remotehost "pwd && ls"
```
## heredoc style
Typically used in a script.
```bash
ssh user@remotehost /bin/bash <<'EOF'
pwd
ls
EOF
```

> 📝 **Note**  
> executing the above without /bin/bash may result in the warning  
> _Pseudo-terminal will not be allocated because stdin is not a terminal_.  
> EOF is surrounded by single-quotes, turning off local variable
> interpolation, the text will be passed as-is to ssh.

## Run command with sudo**

```bash
ssh -t user@remotehost 'pwd; sudo ls'
```
For more complex operations it is easiest to write a script file
locally, then execute it on remote host.

## Execute local script on remote host**

```bash
ssh user@remotehost 'bash -s' < localscript
# or
cat localscript  | ssh user@remotehost
# or
ssh user@remotehost "$(< localscript)"
```

The last one allows interactive commands e.g. sudo

If the local script accepts arguments:

```bash
ssh user@remotehost 'bash -s' -- < localscript arg1 arg2 --argwithdash(es)
```

The double dash after `bash -s` tells bash there are no more arguments -
to bash!

## sudo in heredoc

For example, this works:
```bash
ssh -t example.com "sudo /etc/init.d/apache2 reload"
```
But not this:
```bash
ssh -t example.com <<EOF
sudo /etc/init.d/apache2 reload
EOF

sudo: no tty present and no askpass program specified
```
Solution: store the here document in a variable.
```bash
heredoc="$(cat <<EOF
sudo /etc/init.d/apache2 reload
EOF
)"

ssh -t example.com "$heredoc"
```
A more realistic example:  
- List the ports used by Docker/Portainer on a remote host
- some of your Portainer stacks don't run all the time
- so you can't simply do `docker ps -a`, that only shows the ports for **running** containers

You need to run `find -exec awk ...` as root.
- There's a bunch of single and double quotes involved,
- These are difficult to deal with, since the command is parsed twice,  
  first on your host, then on the remote host.
- So you want to use a heredoc to avoid escaping the quotes

```bash
#!/bin/bash

heredoc="$(cat <<'EOF'
sudo find /var/lib/docker/volumes/portainer_data/_data/ \
-name docker-compose.yml -exec \
awk '
    /container_name:/   { printf "\n"; print }
    /ports:/            { print; getline
                          while($0 ~ "      -") {
                            print; getline
                          }
                        }
' {} +
EOF
)"

ssh -t example.com "$heredoc"
```
Result:
```
[sudo] password for some_user:
    container_name: jellyfin
    ports:
      - 8096:8096
      - 8920:8920 #optional
      - 7359:7359/udp #optional
      - 1900:1900/udp #optional

    container_name: heimdall
    ports:
      - 80:80
#      - 443:443

    container_name: swag
    ports:
      - 443:443
#      - 80:80

    container_name: guacd
    ports:
      - 4822:4822

    container_name: guacamole
    ports:
      - 8080:8080

    container_name: mariadb
    ports:
      - 3306:3306
Connection to example.com closed.
```