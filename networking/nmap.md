# nmap
Scan network devices, report open ports.

> :memo: Note: It may be against your ISP’s terms of
service to use some of Nmap’s more aggressive scan features. 

    # Network scan
    # (older versions) nmap -sP 192.168.1.0/24
    nmap -sn 192.168.1.0/24
    sudo nmap -sn 192.168.1.0/24 (gets MAC addr and NIC mfg.)
    nmap -sn 192.168.1.1-20 (range)
    nmap -sn '192.168.1.*'
    nmap 192.168.1.0/24 --exclude 192.168.1.5,192.168.1.254

    # Host scan
    nmap <ip>
    nmap -F <ip>      # fast
    sudo nmap -O <ip> # detect OS
    nmap -sV <ip>     # detect services and versions
    nmap -sU <ip>     # detect UDP services

    # Specific ports
    nmap -p 80 192.168.1.1      # TCP port 80
    nmap -p T:80 192.168.1.1    # Same
    suso nmap -sU -p U:53 192.168.1.1    # UDP port 53
    sudo nmap -sU -sT -p U:53,137,T:21-25,80,139 192.168.1.254  # multiple ports

    # Alternative host discovery
    nmap -PS <ip>     # TCP SYN scan
    nmap -PA <ip>     # TCP ACK scan
    nmap -PO <ip>     # IP ping
    nmap -PU <ip>     # UDP ping

    # Alternative service discovery
    nmap -sS <ip>      
    nmap -sT <ip>
    nmap -sA <ip>
    nmap -sW <ip>

    # Checking firewalls
    nmap -sN <ip>
    nmap -sF <ip>
    nmap -sX <ip>
