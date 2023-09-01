# IPv6

- Because IPv4 addresses have 32bits, there is a limit of about 4.3
  billion addresses.
- Large chunks are reserved for special use, so there are only about 3.7
  billion globally unique addresses.
- To cope with this, NAT (Network address translation) was developed,
  allowing multiple devices to use a single globally unique address.
- NAT has many problems, though, so IPv6 was developed in 1998.
- IPv6 is intended to re-emphasize the **end-to-end** principle that was
  a goal of the early Internet.

## IPv6 Address Representation

- IPv6 addresses are **128 bits** long. Enough for 100 addresses for
  each **atom** on the surface of Earth.
- Notation of xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx where x is
  case-insensitive hexadecimal character. A hexadecimal character
  represents 4 bits.
- So, 8 groups of 4 hexadecimal characters, each hex char is 4 bits: 8 x
  4 x 4 = 128

Example:

    2031:0000:130f:0000:0000:09c0:876a:130b

There are ways to make this shorter:

1.  Leading zeroes in a group are optional, so you can shorten `:09c0:`
    to `:9c0:`
2.  A group of zeroes can be shorten to one zero. Shorten `:0000:` to
    `:0:`
3.  Continuous groups of zeroes can be shortened. `:0000:0000:` to `::`
    But only once inside an address!

Using these rules, the previous example can be shortened like this:

    2031:0000:130f:0000:0000:09c0:876a:130b
    is identical to
    2031:0:130f::9c0:876a:130b

## IPv6 Address Structure

**Unicast addresses** are divided to two major fields

- first 64bits - Network Prefix
- second 64bits - Interface ID

This is notated as /64 in CIDR notation. The /64 refers to the Prefix.
Most equipment does not support a network prefix of more than 64 bytes.
In general, IPv6 subnets have 64 bits for hosts, allowing for 4.3
billion times the entire IPv4 address space, per subnet.

**Multicast addresses** are divided to four major fields

- first 8 bits are all ones or “FF”. All multicast IPv6 addresses start
  as “FF……”.
- Followed by 4bit “flag field”
- and 4 bit “scope field”
- and a 112bit “group ID”.

There is **no broadcast address** in IPv6

## Types of IPv6 Unicast Addresses

**Global Unicast Address**

This address type is equivalent to IPv4’s public address. Global Unicast
addresses in IPv6 are globally identifiable and uniquely addressable.

- first 48bits - Global Routing Prefix that is typically assigned by
  ISP.
- next 16bits - A subnet ID, that identify links inside a site
- last 64bits - Interface ID
- Currently as of 2020, the first 3 bytes of a valid global IPv6 address
  must be 001.
- So, all global IPv6 addresses must be in the range of 2000 to 3FFF as
  the beginning 16 bits of the address.

**Link-Local**

Every IPv6 enabled interface must generate a link-local address.

- Always starts with FE80.
- The next 48-bits are set to 0
- last 64bits - Interface ID

Link-local addresses are used for communication among IPv6 hosts on a
link (broadcast segment) only. These addresses are not routable, so a
Router never forwards these addresses outside the link.

**Unique-Local Address**

This type of IPv6 address is "in between" the Global and Link-local
addresses communication. These addresses are routable, but not intended
to be Public Addresses, they are not routed over the internet. Similar
to IPv4 10.0.0.0, 192.168.0.0 networks.

- Always starts with FD
- next 40 bits - Routing Prefix
- next 16 bits - Subnet ID
- last 64bits - Interface ID

## Reserved Addresses

- As in IPv4, we use the "slash" notation to indicate how many bits are
  in the netmask.
- For example, in IPv4, you commonly see 192.168.1.0 with a netmask
  255.255.255.0.
- This is sometimes written as 192.168.1.0/24. It means "the first 24
  bits are the network number, the last 8 are for hosts".

IPv6 has reserved a few addresses and address notations for special
purposes.

- ::/128 - unspecified address. Equal to 0:0:0:0:0:0:0:0/128. This is
  equivalent to IPv4 0.0.0.0 with netmask 255.255.255.255, or 0.0.0.0/32
- ::/0 - default route. Equal to 0:0:0:0:0:0:0:0/0. This is equivalent
  to IPv4 0.0.0.0 with netmask 0.0.0.0, or 0.0.0.0/0
- ::1/128 - loopback address. Equal to 0:0:0:0:0:0:0:1/128. Loopback
  addresses in IPv4 are represented by 127.0.0.1 to 127.255.255.255
  series.
- But in IPv6, only ::1/128 represents the Loopback address.

## Assigning IPv6 Addresses

In IPv4 you generally have 2 choices:

- static
- DHCP

There is also "zeroconf", which will auto-config an interface in the
169.254.0.0/16 range. This is intended for a quick way to set up a local
network with no configuration.

By default, a network interface will have **one** IPv4 address. You can
assign more than one address, but usually this is a temporary thing or
part of a load balancing scheme.

IPv6 also supports static, DHCP and a "zeroconf" equivalent called
SLAAC.

- IPv6 is designed to support multiple addresses per interface
- All IPv6 interfaces are required to have a "link-local" address, which
  begins with fe80:
- Typically you will also see one or more "global" addresses (routable
  over the internet).

### SLAAC

Stateless Address Autoconfiguration

- First, the interface generates a unique link-local address (begins
  with fe80:)
- It then sends a Router Solicitation (RS) message to the all-routers
  multicast address FF02::2
- This is part of the Neighbor Discovery Protocol (NDP)
- The router sends a Router Advertisement (RA) to the all-nodes
  multicast address FF02::1
- This RA contains the 64 bit Network Prefix
- The Interface then auto-generates the other 64 bits of the IPv6
  address
- The RA also contains other information, such as the default gateway,
  and recently, the DNS servers

Privacy Extensions:

- Originally, the last 64 bits of an IPv6 address was derived from the
  device's MAC address.
- This is known as EUI-64 Interface ID.
- However, this scheme reveals the MAC address.
- Due to privacy concerns, a hash is used used to generate the Interface
  ID, which bears no resemblance to the MAC address.
- Also, the global IPv6 address is temporary and is regenerated
  periodically (often daily)
- Old temporary addresses are cached for a while, then forgotten.

### DHCP

Two types of DHCP6. Stateful and stateless. Why 2 types? First consider
stateless

Stateless:

- Originally SLAAC did not provide DNS server info.
- And the clients of the time naturally did not try to get DNS servers
  from SLAAC
- Stateless DHCP is used to provide the DNS info, while still
  maintaining the "auto-config" idea.

Stateful:

- This is much like DHCP4
- The administrator has control over address assignment
- Additional parameters can be assigned, above and beyond DNS

However, some devices, such as Android devices, do not support DHCP6
