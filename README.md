# iPXE Boot from BMC (Finally!)

## What?

Most white box switch boxes (at least the ones that ship with [SONiC](https://github.com/Azure/sonic)) has a BMC that is connected to a switch which also connects to external management and the COMex mgmt NIC.

In theory, we can run netboot COMex from BMC. netboot is quite simple, need a dhcp server (typically well known `dnsmasq`)

So the idea is, to boot COMex without an external media i.e. USB stick.

## How?

First we need `dnsmasq` to run on BMC, in my case, AST2500 running [OpenBMC](https://github.com/openbmc/openbmc) thus, `linux-armv5` target.

`dnsmasq` is quite simple to cross-compile especially with [dockcross](https://github.com/dockcross/dockcross/)

```bash
# get dockcross
docker run --rm dockcross/linux-armv5 > ./dockcross-linux-armv5
chmod +x dockcross-linux-armv5
./dockcross-linux-armv5 bash

# build dnsmasq
git clone https://github.com/imp/dnsmasq
cd dnsmasq
make
```

### Selecting iPXE image

YAY! ArchLinux FTW \o/

- :white_check_mark: ArchLinux https://archlinux.org/releng/netboot/
- :white_check_mark: netboot.xyz https://boot.netboot.xyz/ipxe/netboot.xyz.efi
- :x: Ubuntu Netboot http://archive.ubuntu.com/ubuntu/dists/bionic-updates/main/installer-amd64/current/images/netboot/netboot.tar.gz (would need an UEFI image)

### `dnsmasq` config

Make sure to add dhcp advertised gateway to the `eth0`.

```
interface=eth0,lo
bind-interfaces
domain=my.lan
# DHCP range-leases
dhcp-range=eth0,192.168.15.5,192.168.15.253,255.255.255.0,1h

# PXE file to boot from
dhcp-boot=ipxe.efi

# For Ubuntu Netboot Options
# pxe-prompt="Press F8 for menu.", 60
# pxe-service=x86PC, "PXE boot from network server 192.168.15.1", pxelinux

# Gateway
# dhcp-option=3,192.168.15.1
# DNS
dhcp-option=6,1.1.1.1, 8.8.8.8
server=1.1.1.1
# Broadcast Address
# dhcp-option=28,10.0.0.255
# NTP Server
# dhcp-option=42,0.0.0.0

# TFTP Server
dhcp-option=66,192.168.15.1
enable-tftp
tftp-root=/var/lib/tftpboot
log-dhcp
log-queries
log-facility=/tmp/dnsmasq.log
```

and just run the dnsmasq with this,

```bash
kill `pidof dnsmasq`; \
dnsmasq -C dnsmasq.conf && \
tail -f /tmp/dnsmasq.log
```
