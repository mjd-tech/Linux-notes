# DLNA/UPnP

## Simple Service Discovery Protocol (SSDP)

- advertisement and discovery of network services and presence
  information.
- text-based protocol based on HTTPU (HTTP over UDP)
- UDP port number 1900
- IPv4 multicast address 239.255.255.250
- IPv6 uses the address set ff0X::c for all scope ranges indicated by X

This results in the following well-known practical multicast addresses
for SSDP:

    239.255.255.250 (IPv4 site-local address)
    [FF02::C] (IPv6 link-local)
    [FF05::C] (IPv6 site-local)
    [FF08::C] (IPv6 organization-local)
    [FF0E::C] (IPv6 global)

## Event Server

- Enabled through the "Allow programs on other systems to control Kodi"
  option.
- It runs by default on port UDP 9777

## JSON-RPC

- a HTTP- and/or raw TCP socket-based interface for communicating with
  Kodi
- TCP port 9090 (this port can be configured in the advanced settings
  file)
