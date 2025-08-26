# Remote Commands
When running these, ssh exits with the status of the last remote
command, or with status 255 if ssh could not execute the remote command.

> 📝 **Note**  
> These examples assume `remotehost` is defined in your local ~/.ssh/config  
> Otherwise use `ssh user@remotehost`

## Basic Examples

```bash
# Single command
ssh remotehost 'cat /etc/hosts'                # OK
ssh remotehost cat /etc/hosts                  # OK - don't need quotes here - no spaces in filename

ssh remotehost cat 'filename with spaces'      # FAILS
ssh remotehost cat "'filename with spaces'"    # OK - quote the quotes
ssh remotehost cat '"filename with spaces"'    # OK
ssh remotehost cat \'filename with spaces\'    # OK - escape the quotes
ssh remotehost cat \"filename with spaces\"    # OK

ssh remotehost .local/bin/myscript             # path relative to remote $HOME
ssh remotehost /opt/stuff/myscript             # absolute path

# Multiple commands
ssh remotehost 'cd /some/dir; ls'              # commands must be quoted
# or
ssh remotehost "pwd && ls"

# Interactive commands
ssh -t remotehost htop                         # run interactive command
ssh -t remotehost sudo fdisk -l                # run sudo command

# Capture result of interactive command
ssh -t remotehost fzf | tee result.txt         # OK - result written to local file
result=$(ssh -t remotehost fzf)                # FAILS - remote command "hangs"
                                               # variable assignment "eats" stdout
```

> 📝 **Note**  
>  See the section "Quoting woes" below for more information.

## "heredoc" method
 - execute several commands without cramming them all on one line
- can use remote host's environment variables
- can specify filenames containing spaces, without additional quoting tricks

```bash
ssh remotehost /bin/bash <<'EOF'
echo $HOSTNAME
cat "$HOME/Documents/filename with spaces"
EOF
```

> 📝 **Note**  
> executing the above without /bin/bash may result in the warning  
> _Pseudo-terminal will not be allocated because stdin is not a terminal_.  
>   
> EOF is surrounded by single-quotes, turning off local variable
> interpolation, $HOSTNAME and $HOME are those on the remote host.

## sudo in "heredoc" method

For example, this works:
```bash
ssh -t remotehost "sudo /etc/init.d/apache2 reload"
```
But not this:
```bash
ssh -t remotehost <<'EOF'
sudo /etc/init.d/apache2 reload
EOF

sudo: no tty present and no askpass program specified
```
Solution:  
use `cat` and "command substitution"
```bash
ssh -t remotehost "$(cat <<'EOF'
sudo /etc/init.d/apache2 reload
ls 'file with spaces'
echo "$USER on $HOSTNAME"
EOF
)"
```
- This passes the multi-line command as a single string.
- You don't need to do any additional quoting tricks within the heredoc.
- $USER and $HOSTNAME are those on the remote host.

## Quoting woes
- remote ssh commands are parsed TWICE.  First locally, then remotely
- Parsing removes quotes
- To pass quoted strings to the remote host, you have to add or escape quotes locally.
- It can take a lot of trial and error to get this right.

The **heredoc** method avoids this problem.

Another way is to write a script file locally, then execute it on remote host.

## Execute local script on remote host

> 📝 **Note**  
> The most straightforward thing to do is copy your script to the remote host  
> and run it using the "Single command" method above

```bash
# non-interactive script
ssh remotehost 'bash -s' < localscript
# or
cat localscript  | ssh remotehost

# non-interactive script that accepts arguments
ssh remotehost 'bash -s' -- < localscript arg1 arg2
# The double dash after `bash -s` means there are no more arguments to the remote bash.

# interactive script
ssh -t remotehost "$(< localscript)"

# interactive script that accepts arguments - base64 method
ssh -t remotehost "bash <(base64 --decode <<< '$(base64 < localscript)' ) arg1 arg2"
```

The base64 method prevents a lot of problems with more complex interactive scripts.

How the base64 method works:
- `localscript` is base64 encoded into a string, before ssh connection is established.
- On remotehost, this string given to `base64 --decode` as a bash `<<<` "here string"
- The decoded script is then given to bash as a 'piped filename' using `<(...)` "process substitution"
- Although `$(base64 < localscript)` is enclosed in single quotes, it runs, 
  Apparently, single quoted command substitution works in a "double parse" scenario.
- Output can be captured using the `| tee result.txt` method

Reference: https://antofthy.gitlab.io/info/apps/ssh_remote_commands.txt


## A more complex example
- List the ports used by Docker/Portainer on a remote host,
  whether the containers are running or not.
- need to run `find -exec awk` as root

```bash
#!/bin/bash

ssh -t remotehost "$( cat <<'EOF'
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
```json
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