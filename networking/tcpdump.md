# tcpdump

    # Filter port
    tcpdump port 80
    tcpdump src port 1025 
    tcpdump dst port 389
    tcpdump portrange 21-23

    # Filter source or destination IP
    tcpdump src 10.0.0.1
    tcpdump dest 10.0.0.2

    # Filter  everything on network 
    tcpdump net 1.2.3.0/24

    # Logically operators
    tcpdump src port 1025 and tcp 

    # Provide full hex dump of captured HTTP packages
    tcpdump -s0 -x port 80

    # Filter TCP flags (e.g. RST)
    tcpdump 'tcp[13] & 4!=0'
