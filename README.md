# Simple router config for CentOS 8.x

This repository contains configuration for a simple natting router,
using dnsmasq and nftables.

## Philosophy

Keep it simple.  The aim is to offer as simple config as reasonable,
which one can use as the base of her own development.

## Prerequisities

A freshly installed CentOS 8.x machine is expected with one configured
Ethernet device, and with at least one unconfigured one.

## Using the repository

The repository contains the config files, which one should modify for
her needs and copy under /etc.  For some files there will be two
functionally identical versions, one with comments and the other
without.  For heavily commented files, like /etc/dnsmasq.conf, the one
without comments might be more clear.

This file, README.md, contains the instructions for network and
systemd configuration from command line.



