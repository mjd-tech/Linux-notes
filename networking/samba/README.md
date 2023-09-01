# Samba
File Sharing to Windows (and other) systems, using SMB protocol, AKA "Windows File Sharing"  

These instructions intended for a **single user** Linux **Desktop** system.

> :warning: For Linux to Linux file sharing, Prefer **SSH/SFTP** over Samba


## Share types
1. Guest - Anyone can get to it without username/password.
2. Authenticated - needs a username/password
3. Mixed - Guest read-only. Only authenticated user(s) can write. See Windows Quirks below.

## Discovery
This means "Click on Network, and it shows the computers on my network"  
This happens automatically, without any configuration.

To make your Linux host "show up" in Windows 7 and up,
you need **wsdd** (Web Service Discovery Daemon) https://github.com/christgau/wsdd  
Without this, Windows clients have to specify the IP address `\\192.168.1.x\Public`  
Linux clients detect Samba via mDNS (provided avahi is running on the host, it probably is).

Sometimes "Discovery" doesn't work as expected and you have to manually enter sharenames.

Windows client:
`\\hostname\sharename` or `\\hostname.local\sharename` or `\\192.168.1.x\sharename`

Linux client:
`smb://hostname/sharename` or `smb://hostname.local/sharename` or `smb://192.168.1.x/sharename`

## Windows quirks
- Windows automatically sends the current user/password when you click on a share.
- This can be a problem with "Mixed" shares that allow both guests and Authenticated users.
- You may need to use "Map Network Drive" and "Connect using different credentials"
- Windows does not allow you to connect to the same host with different credentials.
- However you can be one user and a guest at the same time.

## User accounts
- Samba maintains its own database of users and passwords.
- Each Samba user is mapped to a Linux user of the same name.
