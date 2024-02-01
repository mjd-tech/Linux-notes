# Wireguard VPN
How to set up wireguard on a Linux box, for VPN access to a home network.

Reference: https://engineerworkshop.com/blog/how-to-set-up-wireguard-on-a-raspberry-pi/

**NOTE:**  
- You cannot be on a remote 192.168.1.0 network and VPN into your home 192.168.1.0 network.
- The IP networks **must be different**
- Change your home network to something other than the common 192.168.0.0 or 192.168.1.0  
- In this example, the home network is 192.168.200.0


## Dynamic DNS
Unless you have a Static IP address from your ISP, you need Dynamic DNS.  
There are numerous ways to do this, which are not covered here.  
In this example, the Dynamic DNS address is `foobar.duckdns.org`

## Enable IP Forwarding on the wireguard Server
```bash
sudo nano /etc/sysctl.conf

# Uncomment this line:
net.ipv4.ip_forward=1

# Enable it
sudo sysctl -p
```

## Install wireguard

```bash
# become root. All configuration must be done as root.
sudo -i
apt install wireguard

# generate keys, save them in files
cd /etc/wireguard
umask 077
wg genkey | tee server_private_key | wg pubkey > server_public_key
wg genkey | tee client_private_key | wg pubkey > client_public_key
```

You will put the content of the key files into the wireguard config file.  
The key files are just text files, you can view them with `cat`

## server config file
Note: in this example the network interface is `enp1s0`  
Run command: `ip -br link` to check.  

```bash
nano /etc/wireguard/wg0.conf
```
Note:
- Replace `SERVER_PRIVATE_KEY_HERE` with the contents of `server_private_key`
- Replace `CLIENT_PUBLIC_KEY_HERE` with the contents of `client_public_key`

To avoid tedious copy/pasting, you can tell nano to insert the contents of another file, using Ctrl-R `filename`  
If you're using vim,  press :r `filename`

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

**NOTES:**  
> - Always **stop** wireguard before editing the wg0.conf file.
> - Do **NOT** edit wg0.conf while wireguard is running

"Address" is an arbitrary IP network that isn't being used on either the client or server side. Use a "private" IP address in the range:

- 10.0.0.0 to 10.255.255.254
- 172.16.0.0 to 172.31.255.254
- 192.168.0.0 to 192.168.255.254

"Listen Port" is changed from the default 51820 for "security by obscurity"

Wireguard will dynamically add `Endpoint = ...` to the `[Peer]` section when you connect to it.  

## Autostart WireGuard at boot
Also, ensure file permissions are correct.
```bash
systemctl enable wg-quick@wg0
chown -R root:root /etc/wireguard/
chmod -R og-rwx /etc/wireguard/*
```

## Stopping, starting wireguard service
Must be done as root, use sudo if you're not root.

```bash
systemctl stop wg-quick@wg0
systemctl start wg-quick@wg0
# You don't have to be root to check status
systemctl status wg-quick@wg0
```

## Port Forwarding on your home Router
- in this example we used 43210 as "Listen Port"
- forward port 43210 UPD to the ip address of the wireguard server

## Client config
Typically the client config will be a Gui. There are too many Gui clients to cover here.
Here's how to test the connection, on Linux, with cli.

`sudo nano /etc/wireguard/wg0.conf`

Put this, replacing Keys and AllowedIPs
```
[Interface]
Address = 10.253.3.2/32
PrivateKey = CLIENT_PRIVATE_KEY_HERE

[Peer]
PublicKey = SERVER_PUBLIC_KEY_HERE
Endpoint = foobar.duckdns.org:43210
AllowedIPs = 192.168.200.0/24
```

- Test connection with `sudo wg-quick up wg0`
- Close connection with `sudo wg-quick down wg0`

**NOTES:**
- If you set AllowedIPs to 0.0.0.0/0, all traffic from the client goes through the VPN.
- Do not set a "default gateway" in gui client.

## Accessing your hosts
Wireguard is a layer 3, routed connection. It's not the same as actually "being there".  
Clicking on a "Network" icon won't show your home hosts.  
Any "discovery services" that use Multicast won't work. ie. DLNA

You get to your home hosts by **IP address**, or by hostname in the `/etc/hosts` file on your **client**.  
You could configure a home DNS server and set wireguard to use that, but that's not covered here.  
Using a home DNS server won't make "discovery services" work.

## Adding more clients
```bash
sudo -i
cd /etc/wireguard
```

Create keys for new client:
```bash
umask 077
wg genkey | tee client2_private_key | wg pubkey > client2_public_key
```

edit /etc/wireguard/wg0.conf

- add another `[Peer]` section similar to the one above.
- but change "allowed ips" to 10.253.3.3/32, in other words one higher than the previous Peer.
- Use client2-public key.

client config similar to the first client. But with different "Address" and "PrivateKey"

## Troubleshooting
Wireguard does not log by default.  
Enable logging:
```bash
echo 'module wireguard +p' | sudo tee /sys/kernel/debug/dynamic_debug/control
```
This will write WireGuard logging messages to the kernel log, which can be watched live with:
```bash
sudo dmesg -wT
```
Disable logging:
```bash
echo 'module wireguard -p' | sudo tee /sys/kernel/debug/dynamic_debug/control
```
