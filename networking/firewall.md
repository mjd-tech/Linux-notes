# Firewall
Desktop - ufw; server - iptables
## IPv4/IPv6 packet filtering and NAT – "iptables"

- For IPv6 use: `ip6tables`
- Print all rules: `iptables -S `, `iptables -L -v`
- Clear all configured rules: `iptables -F`
- Basic chains: `iptables -L | grep policy` *…* INPUT, FORWARD, OUTPUT
- Accept connection on port *N*: `iptables -A input -p tcp -dport N` -j ACCEPT
- Accept connection from IP: `iptables -A input -p tcp -dport N -s
  IP/mask -j ACCEPT`
- Drop connection from `192.168.10.x`: `iptables -A INPUT -s
  192.168.10.0/24 -j DROP`
- Enable SSH: `iptables -A INPUT -m tcp -p tcp --dport 22 -j ACCEPT`
- Enable SSH, HTTP, HTTPS: `iptables -A INPUT -p tcp -m state --state NEW -m multiport --dports ssh,http,https -j ACCEPT`
- Save iptables: `/sbin/iptables-save > /etc/sysconfig/iptables`, `/etc/init.d/iptables save`
- Network Address Translation (NAT) / Masquarage: `iptables -t nat -A POSTROUTING -s 10.200.0.0/24 -o eth0 -j MASQUERADE`
- Delete rule: `iptables -t nat --line-numbers -L` (list in table); `iptables -t nat -D PREROUTING 2` (delete 2nd line)

## Dynamic Firewall Manager – "firewalld"

- Check status: "firewall-cmd --state", "systemctl status firewalld"
- Print all rules: "firewall-cmd --list-all"
- List zones: "firewall-cmd --get-active-zones", "firewall-cmd --get-zones"
- Get or set default zone: "firewall-cmd --get-default-zone", "--set-default-zone=*ZONE*"
- Set default zone: "firewall-cmd --set-default-zone=*ZONE*"
- Without "--permament" option any changes will not be available after restart.
- Open TCP port in zone: "firewall-cmd --permanent --zone=*ZONE* --add-port=8080/tcp"
- Enable services: "firewall-cmd --permanent --add-service={http,https"
- Activate changes in configuration: "firewall-cmd --reload"
- Disable: "--remove-port=*port/protocol*", "--remove-service=*service*", "--remove-source=*X.X.X.X/Y*"
- Network Address Translation (NAT) / Masquarage: "firewall-cmd --zone=external --add-masquerade"
- Forward packets to other IP and port: "firewall-cmd --zone=external --add-forward-port=port=22:proto=tcp:toport=2055:toaddr=192.0.2.55"
- Rich language examples:
  - "firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source
    address=172.25.X.10/32 service name="http" log level=notice
    prefix="NEW HTTP " limit value="3/s" accept '"
  - "firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source
    address=10.0.0.1/32 forward-port port=443 protocol=tcp to-port=22'"

