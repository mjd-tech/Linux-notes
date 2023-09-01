# Wireguard VPN
How to set up wireguard on a Linux box, for VPN access to a home network.

**NOTE:**  
You cannot be on a remote 192.168.1.0 network and VPN into your home 192.168.1.0 network.  
The IP networks must be different.  
Change your home network to something other than the common 192.168.0.0 or 192.168.1.0  
In this example, the home network is 192.168.200.0

Reference: https://engineerworkshop.com/blog/how-to-set-up-wireguard-on-a-raspberry-pi/

## Dynamic DNS
Unless you have a Static IP address from your ISP, you need Dynamic DNS.  
There are numerous ways to do this, which are not covered here.
In this example, the Dynamic DNS address is foobar.duckdns.org

## Install wireguard

    # become root. All configuration must be done as root.
    sudo -i
    apt install wireguard

    # generate keys, save them in files
    cd /etc/wireguard
    umask 077
    wg genkey | tee server_private_key | wg pubkey > server_public_key
    wg genkey | tee client_private_key | wg pubkey > client_public_key

You will paste the content of the key files into the wireguard config file.  
The key files are just text files, you can view them with `cat`

## Enable IP Forwarding on the wireguard Server

    nano /etc/sysctl.conf
    
    # Uncomment this line:
    net.ipv4.ip_forward=1
    
    # Enable it
    sysctl -p

## server config file

    nano /etc/wireguard/wg0.conf

Note: in this example the network interface is `enp1s0`  
Run command: `ip -br link` to check.  

```
[Interface]
Address = 10.253.3.1/24
SaveConfig = true
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o enp1s0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o enp1s0 -j MASQUERADE
ListenPort = 43210
PrivateKey = SERVER_PRIVATE_KEY_HERE

[Peer]
PublicKey = CLIENT_PUBLIC_KEY_HERE
AllowedIPs = 10.253.3.2/32
```

**NOTE:**  
"Address" is an arbitrary IP network that isn't being used on either the client or server side.  
"Listen Port" is changed from the default 51820 for "security by obscurity"  
Wireguard will dynamically add `Endpoint = ...` to the `[Peer]` section when you connect to it.  

## Autostart WireGuard at boot
Also, ensure file permissions are correct.

    systemctl enable wg-quick@wg0
    chown -R root:root /etc/wireguard/
    chmod -R og-rwx /etc/wireguard/*

## Port Forwarding on your home Router
- in this example we used 43210 as "Listen Port"
- forward port 43210 UPD to the ip address of the wireguard server

## Client config
Typically the client config will be a Gui. There are too many Gui clients to cover here.
Shown here is what you would use for cli.

    [Interface]
    Address = 10.253.3.2/32
    PrivateKey = CLIENT_PRIVATE_KEY_HERE

    [Peer]
    PublicKey = SERVER_PUBLIC_KEY_HERE
    Endpoint = foobar.duckdns.org:43210
    AllowedIPs = 192.168.200.0/24

**NOTES:**
- If you set AllowedIPs to 0.0.0.0/0, all traffic from the client goes through the VPN.
- Do not set a "default gateway" in gui client.

## Accessing your hosts
Wireguard is a layer 3, routed connection. It's not the same as actually "being there".  
Clicking on a "Network" icon won't show your home hosts.  
Any "discovery services" that use Multicast won't work. ie. DLNA

You get to your home hosts by **IP address**, or by hostname in the `/etc/hosts` file on your client.  
You could configure a home DNS server and set wireguard to use that, but that's not covered here.  
Using a home DNS server won't make "discovery services" work.

## Adding more clients

    sudo -i
    cd /etc/wireguard

Create keys for new client:

    umask 077
    wg genkey | tee client2_private_key | wg pubkey > client2_public_key

edit /etc/wireguard/wg0.conf

- add another `[Peer]` section similar to the one above.
- but change "allowed ips" to 10.253.3.3/32, in other words one higher than the previous Peer.
- Use client2-public key.

client config similar to the first client. But with different "Address" and "PrivateKey"


