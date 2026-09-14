---
layout: post
title: "DHCPv6 Cloned VM - Ubuntu"
date: 2026-08-13 20:26:25 +0000
tags: ["dhcp", "ipv6", "Ubuntu"]
blogger_orig_link: https://lexsysko.blogspot.com/2026/08/dhcpv6-cloned-vm-ubuntu.html
---

The main problem with a cloned VM is that the DUID doesn’t change, which causes issues when trying to get a new DHCPv6 address.
  
  

### Netplan

```
dhcp6: true
dhcp-identifier: mac
```

/etc/netplan/00-installer-config.yaml  

```
network:
  ethernets:
    ens18:
      addresses:
      - 10.10.10.106/24
      mtu: 9000
      dhcp6: true
      dhcp-identifier: mac
      match:
        macaddress: bc:24:11:12:11:3b
      nameservers:
        addresses:
        - 10.10.10.1
        search:
        - example.net
      routes:
      - to: default
        via: 10.10.10.1
      set-name: ens18
  version: 2
```

### DUIDType or Reset Machine ID

Instead of clearing the unique machine identifier, you can set it up to use the link-layer MAC for DHCPv6.

```
sudo mkdir -p /etc/systemd/networkd.conf.d 
sudo nano /etc/systemd/networkd.conf.d/10-dhcp6-duid.conf
```

```
[DHCPv6]
   DUIDType=link-layer
```

Check the result:

```
sudo systemctl restart systemd-networkd 
networkctl status ens18
...
DHCPv6 Client DUID: DUID-LL:0001bc241112113b
```

### Reset Machine ID

Clear the unique machine identifier so a new one generates on next boot:

```
sudo rm -f /etc/machine-id
sudo systemd-machine-id-setup
```
