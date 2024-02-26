# Remote Commands
When running these, ssh exits with the status of the last remote
command, or with status 255 if ssh could not execute the remote command.

## Single command

```bash
ssh user@remotehost 'cat /etc/hosts'                # OK
ssh user@remotehost cat /etc/hosts                  # OK - don't need quotes here
ssh user@remotehost cat 'filename with spaces'      # FAILS
ssh user@remotehost cat "'filename with spaces'"    # OK - quote the quotes
ssh user@remotehost cat \'filename with spaces\'    # OK - escape the quotes
```

> 📝 **Note**  
>  See the section "Quoting woes" below for more information.

## Multiple commands

```bash
ssh user@remotehost 'pwd; ls'
# or
ssh user@remotehost "pwd && ls"
```
## "heredoc" method
This allows you to execute several commands without cramming it all on one line.
Can use remote host's environment variables, and specify filenames containing spaces, without additional quoting tricks.

```bash
ssh user@remotehost /bin/bash <<'EOF'
echo $HOSTNAME
ls "$HOME/Documents/filename with spaces"
EOF
```

> 📝 **Note**  
> executing the above without /bin/bash may result in the warning  
> _Pseudo-terminal will not be allocated because stdin is not a terminal_.  
> EOF is surrounded by single-quotes, turning off local variable
> interpolation, $HOSTNAME and $HOME are those on the remote host.

## Run command with sudo

```bash
ssh -t user@remotehost 'pwd; sudo ls'
```
## Quoting woes
- remote ssh commands are parsed twice.  First locally, then remotely
- Parsing removes quotes
- To pass quoted strings to the remote host, you have to add or escape quotes locally.
- It can take a lot of trial and error to get this right.
- You end up with a different command syntax local vs remote.

The **heredoc** method avoids this problem.

Another way is to write a script file locally, then execute it on remote host.

## Execute local script on remote host

```bash
ssh user@remotehost 'bash -s' < localscript
# or
cat localscript  | ssh user@remotehost
# or
ssh -t user@remotehost "$(< localscript)"
```

The last one allows interactive commands e.g. `sudo`

If the local script accepts arguments:

```bash
ssh user@remotehost 'bash -s' -- < localscript arg1 arg2 --argwithdash(es)
```

The double dash after `bash -s` means there are no more arguments to the remote bash.

## sudo in ssh heredoc

For example, this works:
```bash
ssh -t example.com "sudo /etc/init.d/apache2 reload"
```
But not this:
```bash
ssh -t example.com <<'EOF'
sudo /etc/init.d/apache2 reload
EOF

sudo: no tty present and no askpass program specified
```
Solution:  
use `cat` and "command substitution"
```bash
ssh -t example.com "$(cat <<'EOF'
sudo /etc/init.d/apache2 reload
ls 'file with spaces'
echo "$USER on $HOSTNAME
EOF
)"
```
- This passes the multi-line command as a single string.
- You don't need to do any quote escaping tricks within the heredoc.
- $USER and $HOSTNAME are those on the remote host.

A more complex example:  
- List the ports used by Docker/Portainer on a remote host,
  whether the containers are running or not.
- need to run `find -exec awk` as root

```
#!/bin/bash

ssh -t example.com "$( cat <<'EOF'
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
Connection to example.com closed.
```