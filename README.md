# Simple router config for RHEL/CentOS 8.x

This repository contains configuration for a simple natting IPv4
router, using dnsmasq and nftables.

## Philosophy

Keep it simple.  The aim is to offer as simple config as reasonable,
which one can use as the base of her own development.  Firewalld is
not used, because it is not easy to understand.  Forwarding and
firewall is made by nftables instead.  The instructions are given as a
recipe with newbie friendly tone.

## Prerequisities

A freshly installed RHEL 8.x or CentOS 8.x machine is expected with
one configured Ethernet device, and with at least one unconfigured
one.

## Using the repository

The repository contains the config files, which one should modify for
her needs and copy under /etc.  For some files there will be two
functionally identical versions, one with comments and the other
without.  For heavily commented files, like /etc/dnsmasq.conf, the one
without comments might be more clear.

This file, README.md, contains the instructions for network and
systemd configuration from command line.

# The Step by Step Recipe

## Ethernet

Assuming `eth0` as the WAN device and `eth1` the LAN (local network)
device.  `eth0` should be working already, and `eth1` still
unconfigured.  By default, CentOS does not use the old style `ethX`
names anymore.  The naming here was choosed intentionally to force you
to check your config.

* Use commands
  `sudo nmcli connection modify enp6s0 ipv4.method manual`
  `sudo nmcli connection modify enp6s0 ipv4.address "192.168.1.1/24"`
  `sudo nmcli connection up eth1`
  to configure the LAN device.  `nmcli` will alter the NetworkManager
  config, so the new config will survive the reboot.

* Edit the file `/etc/sysconfig/network-scripts/ifcfg-eth1` and
  replace the line `ONBOOT=no` by `ONBOOT=yes`, otherwise your `eth1`
  will not get up on the next boot.

* Since we are making an IPv4 router, a purist might also ensure the
  above file has options `IPV6INIT=no` and `IPV6_AUTOCONF=no`.  But
  that doesn't really matter.  The file is not synced to
  NetworkManager in real time.  If ones makes changes, the command
  `nmcli reload eth1` will activate them.

## Masquerading NAT

* Create file `/etc/sysctl.d/99-router.conf` with line
  `net.ipv4.ip_forward=1` and give it the correct SEcontext by `sudo
  chcon system_u:object_r:etc_t:s0 /etc/sysctl.d/99-router.conf`.  The
  file should be owned by `root:root`.  Activate it by running `sudo
  sysctl -p`.

* Copy the file `ipv4_nat_fw.nft` to ´/etc/nftables/ipv4_nat_fw.nft`
  (owned by `root:root` and rw permissions for 'root' only).

* Edit the file `/etc/sysconfig/nftables.conf` bu adding the line
  `include "/etc/nftables/ipv4_nat_fw.nft"`.

* Disable firewalld and start nftables by
  `sudo systemctl stop firewalld`
  `sudo systemctl start nftables`
  Now you can check whether `sudo nft list ruleset` gives the rules
  defined at `ipv4_nat_fw.nft`.  If ok, proceed with making nftables
  permanent by
  `sudo systemctl disable firewalld`
  `sudo systemctl enable nftables`.

## DNS server and DCHP

* In order to have working LAN, you need DNS server and DHCP service
  there.  This is done by `dnsmasq`.  The service binds to the port 53
  of the interface defined in the config file.  If you happen to run
  e.g. `libvirtd`, you should disable that first, because in the
  default config in CentOS 8.1 it reserves the port and prevents
  `dnsmasq` from starting.  If you really want to run libvirtd in your
  router, please configure it to bind on its dedicate interface only.

* Install `dnsmasq` by `sudo yum install dnsmasq`.

* Edit the `dhcp-host` lines of the dnsmasq.conf example file to
  resemble your LAN, and copy the file to `/etc/dnsmasq.conf`.  It
  should be owned by `root:dnsmasq` and to ensure the correct
  SEcontext, say
  `sudo chcon system_u:object_r:dnsmasq_etc_t:s0 /etc/dnsmasq.conf`.

* Give the IP-address to name mapping of your LAN machines in `/etc/hosts` with lines like
  `192.168.1.2 machine1.home.localnet machine1`.

