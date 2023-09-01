# OpenVPN Notes

Commercial VPN providers:

- You send all your internet traffic through the provider.
- Internet sites that you connect to will "see" the VPN provider's IP
  address not your real public IP address
- You can connect to VPN servers in different countries.
- Can still access your LAN normally

LAN-to-LAN VPN:

- Connects two or more LANs via the internet
- For example, connect a branch office to the main office.
- Often set up on the routers that connect to the internet.
- Usually "always on"

Host-to-remote LAN:

- Connect a host (pc,laptop,tablet,phone,etc) to a remote LAN via the
  internet.
- The remote LAN needs a VPN server, the host needs a VPN client.
- Need to be aware of the "default gateway".
- VPN can possibly change it so normal internet traffic is routed
  through remote LAN's router instead of yours, you may not want that.

The main considerations for VPN are:

- OpenVPN vs other technology
- Routed vs Bridged
- Will the VPN server run on the "default gateway" (router) or run on
  another device?
- How to handle "default gateway" on clients.

In any VPN situation, you are effectively joining two networks together.

- These need to have **different IP address ranges** or else major
  headaches.
- Home routers typically assign the LAN: 192.168.0.x 192.168.1.x or
  10.0.0.x
- So if you are going to put a VPN server in a home LAN you should
  change the IP address to something uncommon ie 192.168.143.x

## Routed vs Bridged

- Routing passes layer 3 (IP) unicast (not multicast) traffic.
- Bridging passes layer 2 (MAC) traffic, also IP multicast
- Routing is a bit easier to set up
- Bridging is easier for clients to use.
- Bridging is "slower" due to all the broadcast and multicast traffic.

## OpenVPN on the Default Gateway

- Ideally, OpenVPN server would run on the router.
- Most home routers have lame software.
- If its a Linksys, it might have openvpn server, but will only do
  routed mode.
- You can flash some routers with openwrt, but that opens another can of
  worms.

## Routing considerations

                            +-------------------------+
                 (public IP)|                         |
    {INTERNET}=============={     Router              |
                            |                         |
                            |         LAN switch      |
                            +------------+------------+
                                         | (192.168.0.1)
                                         |
                                         |              +-----------------------+
                                         |              |                       |
                                         |              |        OpenVPN        |  eth0: 192.168.0.10/24
                                         +--------------{eth0    server         |  tun0: 10.8.0.1/24
                                         |              |                       |
                                         |              |           {tun0}      |
                                         |              +-----------------------+
                                         |
                                +--------+-----------+
                                |                    |
                                |  Other LAN clients |
                                |                    |
                                |   192.168.0.0/24   |
                                |   (internal net)   |
                                +--------------------+

- Here tun0 is configured as 10.8.0.1 as a VPN, with the whole VPN
  network configured as 10.8.0.0/24.
- VPN clients will be 10.8.0.2-254

First, be sure that IP forwarding is enabled on the OpenVPN server. And
make it persistent.

    sudo -i
    sysctl -w net.ipv4.ip_forward=1
    echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf

In the OpenVPN server config you will need these lines (among others!):

    dev tun0
    topology subnet
    server 10.8.0.0 255.255.255.0
    push "route 192.168.0.0 255.255.255.0"

The Router needs to have **port forwarding** to 192.168.0.10, the IP
address of the OpenVPN on the internal network.

In this state, the remote client will only be able to connect to the
OpenVPN server, but not to the "Other LAN clients".

**Plan A:**

- Go into the router and add a static route for 10.8.0.0/24 to be sent
  via 192.168.0.10.
- This is needed for the traffic from your LAN clients to be able to
  find their way back to the VPN clients.
- If this is not possible, you need add such routes explicitly on all
  the LAN clients you want to access via the VPN.

Now **optionally** set some firewall rules in the OpenVPN server (not
really needed if you are behind a router and doing port forwarding)

    # Allow traffic initiated from VPN to access LAN
    iptables -I FORWARD -i tun0 -o eth0 -s 10.8.0.0/24 -d 192.168.0.0/24 -m conntrack --ctstate NEW -j ACCEPT

    # Allow established traffic to pass back and forth
    iptables -I FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

**Plan B:**

If your router has lame-brain software that does not support static
routes, and you don't want to deal with manually adding static routes to
all your devices on the LAN, you need to do NAT.

The drawback is that all VPN clients look like they come from the VPN
server itself. You will not see the IP address of the VPN client at all.
This approach is generally considered as a last option if proper routing
is not feasible.

Set the following firewall rule in the OpenVPN server:

    iptables -t nat -I POSTROUTING -o eth0 -s 10.8.0.0/24 -j MASQUERADE

**Notes on firewall rules**

- They are not persistent.
- There are various tools to configure firewall in Linux. ufw,
  firewalld, iptables, nftables
- It's all very confusing and annoying.

For now, the easiest thing to do is to put the firewall rules in a shell
script. First, put this in OpenVPN server config

    # set up nat at startup, remove at shutdown
    script-security 2
    up /etc/openvpn/server/up.sh
    down /etc/openvpn/server/down.sh

Then create /etc/openvpn/server/up.sh

    #!/bin/sh
    iptables -t nat -I POSTROUTING -o eth0 -s 10.8.0.0/24 -j MASQUERADE

And /etc/openvpn/server/down.sh

    #!/bin/sh
    iptables -t nat -F

It looks like iptables is going away and the new system is nftables.

- there is a command that translates iptables to nftables:
  iptables-translate
- you give it iptables parameters and it spits out the equivalent
  command for nftables

<!-- -->

    iptables-translate -t nat -I POSTROUTING -o eth0 -s 10.8.0.0/24 -j MASQUERADE
    nft insert rule ip nat POSTROUTING oifname "eth0" ip saddr 10.8.0.0/24 counter masquerade

    iptables-translate -t nat -F
    nft flush table ip nat

So you would use the nft commands instead of the iptables commands in
the up and down shell scripts.

## Bridging

- Use a "tap" interface instead of "tun"
- On Raspberry Pi, you **must use eth0** for bridging. Bridging does not
  work on the wifi adapter wlan0, but routing does work.
- You **must use a static IP address**, outside the range of the
  router's DHCP pool
- The openvpn server has a range of ip addresses that it will assign to
  clients
- This range **must** be outside the router's DHCP pool
- Does not need iptables rules
- You do need a helper shell script to set up the bridge.

Source:
<https://www.emaculation.com/doku.php/bridged_openvpn_server_setup>

openvpn-bridge

``` bash
#!/bin/sh

# If it won't boot properly try this
#sleep 5

# Define Bridge Interface
br="br0"

# Define list of TAP interfaces to be bridged,
# for example tap="tap0 tap1 tap2".
tap="tap0"

# Define physical ethernet interface to be bridged
# with TAP interface(s) above.
eth="eth0"
eth_mac="b8:27:eb:d0:29:27"
eth_ip_netmask="192.168.27.40/24"
eth_broadcast="192.168.27.255"
eth_gateway="192.168.27.254"

case "$1" in
start)
    sudo systemctl stop dhcpcd
    for t in $tap; do
        openvpn --mktun --dev $t
    done

    brctl addbr $br
    brctl addif $br $eth

    for t in $tap; do
        brctl addif $br $t
    done

    for t in $tap; do
        ip addr flush dev $t
        ip link set $t promisc on up
    done

    ip addr flush dev $eth
    ip link set $eth promisc on up

    ip addr add $eth_ip_netmask broadcast $eth_broadcast dev $br
    ip link set br0 address $eth_mac
    ip link set $br up

    ip route add default via $eth_gateway
    ;;
stop)
    ip link set $br down
    brctl delbr $br

    for t in $tap; do
        openvpn --rmtun --dev $t
    done

    ip link set $eth promisc off up
    ip addr add $eth_ip_netmask broadcast $eth_broadcast dev $eth

    ip route add default via $eth_gateway
    sudo systemctl start dhcpcd
    ;;
*)
    echo "Usage:  openvpn-bridge {start|stop}"
    exit 1
    ;;
esac
exit 0
```

Make it executable:

    chmod 744 /etc/openvpn/openvpn-bridge

We need to tell OpenVPN to make use of our “openvpn-bridge” script.

    vi /lib/systemd/system/openvpn@.service

Copy these two lines:

    ExecStartPre=/etc/openvpn/openvpn-bridge start
    ExecStopPost=/etc/openvpn/openvpn-bridge stop

and paste them at the bottom of the \[Service\] section.

Exit and save.

**Note:**

- The example script is based on Raspberry Pi, running Raspberry Pi OS
  (formerly Raspbian)
- It stops and starts the "dhcpcd" service when it creates/removes the
  bridge.
- This is because a new network interface, br0, is added to the system.
- dhcpcd tries to assign an ip addresses to br0, from the router's DHCP
  pool.
- We don't want this, so the easiest way is to just stop the dhcpcd
  service while the bridge is active (while OpenVPN server is running)
- If OpenVPN server is stopped, the dhcpcd service resumes and the Pi
  acts like normal.
- Depending on your system, you may have another network management
  service, such as NetworkManager.
- So you stop/start that service instead.
- If you use purely static setup, then you don't need to stop/start a
  network service, so remove those lines from the script.
- When the br0 interface is setup, it may or may not assume the MAC
  address of eth0.
- This can be a problem with some routers port forwarding rules.
- To fix that, we need to force br0 to assume the MAC address of eth0
  (or whatever the ethernet adapter is called.)
- So this script is modified from the original to account for that.

## SSL keys, certificates, etc.

Some definitions:

- PKI - Public key infrastructure.
- Public Key - used for encrypting data. Distributed to remote clients.
- Private Key - used for decrypting data. Stays on the server.
- Asymmetric keys - different keys used to "lock" and "unlock" data.
  Public and Private keys are asymmetric.
- Symmetric key - same key is used to "lock" and "unlock". This is what
  is actually used once "handshaking" is done with the asymmetric keys.
- Certificate - a public key with a label identifying the owner.
- CA - Certificate Authority - Verifies the validity of certificates.
  Required by PKI, we will create our own.

| Filename    | Needed By                | Purpose                   | Secret |
|-------------|--------------------------|---------------------------|--------|
| ca.crt      | server + all clients     | Root CA certificate       | NO     |
| ca.key      | key signing machine only | Root CA key               | YES    |
| dh{n}.pem   | server only              | Diffie Hellman parameters | NO     |
| server.crt  | server only              | Server Certificate        | NO     |
| server.key  | server only              | Server Key                | YES    |
| client1.crt | client1 only             | Client1 Certificate       | NO     |
| client1.key | client1 only             | Client1 Key               | YES    |

### easyrsa

- There are 2 versions of this; 2.x and 3.x
- The syntax is different
- Here, we will use 3.x, doing all actions as root

The first thing you do - Copy Easy-RSA to OpenVPN's directory:

    cp -r /usr/share/easy-rsa /etc/openvpn
    cd /etc/openvpn/easy-rsa

There is a file named vars.example. Copy this file:

    cp vars.example vars

- Edit the file...
- Find this line: \#set_var EASYRSA_CERT_EXPIRE 825
- change it to: set_var EASYRSA_CERT_EXPIRE 3650
- this changes the number of days when a certificate expires to 10
  years.

Now, initialize.

    ./easyrsa init-pki

- This creates the directory /etc/openvpn/easy-rsa/pki. Stores the keys
  and certificates that you create.
- Do **NOT** run this command again unless you want to delete all your
  existing keys and certificates!

Create a Certificate Authority (CA) by entering

    ./easyrsa build-ca nopass

- The Common Name defaults to "OpenVPN-CA" (without quotes).
- Choose a more descriptive name: "Fred Flintstone VPN-CA"
- This is especially important if you manage more than one VPN server.
- You end up with a lot of ca.crt files and you need a way to tell which
  is which.

Two files get created: a certificate with public key - **ca.crt** and a
private key - **ca.key** (in the "private" directory)

- Both of these are text files than can be viewed with cat or less, but
  for more info, use the openssl commands below.
- examine the contents of the certificate with:
  `openssl x509 -noout -text -in ca.crt`
- examine the contents of the the private key:
  `openssl rsa -noout -text -in ca.key` (not much useful info in a key
  file)

Create the server credentials by entering

    ./easyrsa gen-req servername nopass  # put the actual server name

- The Common Name will be set to the servername by default, so no entry
  is required.
- This creates a request file: servername.req (in the "reqs" directory)
- Also creates a private key: servername.key (in the "private"
  directory)

Sign the server credentials by entering

    ./easyrsa sign-req server servername

- Enter “yes” (without quotes) as requested.
- This creates a certificate for the server: servername.crt (in the
  "issued" directory)

Generate Diffie-Hellman parameters by entering

    ./easyrsa gen-dh

- This creates a file: dh.pem
- The purpose of this will be explained below.

If your clients will be using user/password to connect, you are done.
Otherwise, are using password based login Now we'll create the client
credentials.

To create credentials for a client called “joe”, enter

    ./easyrsa gen-req joe nopass

The Common Name will be set to “joe” by default, so no entry is
required.

- Creates: joe.req (in the "reqs" directory)
- Creates: joe.key (in the "private" directory)

Sign the credentials of client “joe” by entering

    ./easyrsa sign-req client joe

Enter “yes” (without quotes) as requested.

- Creates: joe.crt (in the "issued" directory)

You can make more client credentials by changing “joe” in the previous
two commands. Each client's Common Name must be unique.

### Renew ca.crt Certificate

**This info is unverified**

If your ca.crt expires, you can "renew" the server without having to
distribute new ca.crt files to clients.

You need 2 files from the server. These should be within
/etc/openvpn/easy-rsa, depending on how it was set up.

1.  ca.crt
2.  ca.key

Both files are owned by root. The permissions are usually:

- ca.crt 644
- ca.key 600

Be aware of the owner and permissions if you intend to generate the new
ca.crt as a non-root user.

    openssl x509 -x509toreq -in ca.crt -signkey ca.key -out renewedselfsignedca.csr
    echo -e "[ v3_ca ]\nbasicConstraints= CA:TRUE\nsubjectKeyIdentifier= hash\nauthorityKeyIdentifier= keyid:always,issuer:always\n" > renewedselfsignedca.conf
    openssl x509 -req -days 1095 -in renewedselfsignedca.csr -signkey ca.key -out renewedselfsignedca.crt -extfile ./renewedselfsignedca.conf -extensions v3_ca

    # WF 2017-06-30
    # https://serverfault.com/a/501513/162693
    CACRT=SnakeOilCA.crt
    CAKEY=SnakeOilCA.key
    NEWCA=SnakeOilCA2017
    serial=`openssl x509 -in $CACRT -serial -noout | cut -f2 -d=`
    echo $serial
    openssl x509 -x509toreq -in $CACRT -signkey $CAKEY -out $NEWCA.csr
    echo -e "[ v3_ca ]\nbasicConstraints= CA:TRUE\nsubjectKeyIdentifier= hash\nauthorityKeyIdentifier= keyid:always,issuer:always\n" > $NEWCA.conf
    openssl x509 -req -days 3650 -in $NEWCA.csr -set_serial 0x$serial -signkey $CAKEY -out $NEWCA.crt -extfile ./$NEWCA.conf -extensions v3_ca
    openssl x509 -in $NEWCA.crt -enddate -serial -noout
