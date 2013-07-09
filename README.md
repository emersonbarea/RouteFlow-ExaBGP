# Welcome

This version of RouteFlow ([RouteFlow](http://cpqd.github.io/RouteFlow/)) is responsible for demonstrate the behavior of a topology dynamically routed via BGP, where the Quagga engine ([Quagga](http://www.nongnu.org/quagga/)) is used with ExaBGP ([ExaBGP](https://code.google.com/p/exabgp/)) to route the topology used in this example.

# Overview

To create the proposed environment, besides RouteFlow, Guagga and ExaBGP, it is also required to use Mininet ([Mininet](http://mininet.org/)). It was used the “Four routers running OSPF” example of topology `https://sites.google.com/site/routeflow/documents/tutorial2-four-routers-with-ospf`, where there was only a change in the rfvmA virtual machine, taking off Quagga and using ExaBGP instead.

# Environment

## 1 – createExaBGP

This script creates lxc routers using config/ExaBGP configurations files. RfvmA configuration file creates a lxc that includes `/etc/init.d/exabgp` script that is responsible for start and stop ExaBGP engine.

## 2 – rftestExaBGP

This script is responsible for start the services RouteFlow topology.

After created and started RouteFlow services, Mininet should be used to mount the environment for testing. To verify the correct operation, you just have to examine the routing table of each lxc machine (rfvmA, rfvmB, rfvmC e rfvmD). Note that the route announced by ExaBGP and Quagga are different in rfvmA. For example:

Quagga: publishes 172.31.1.0/24

ExaBGP: publishes 172.31.1.100/32

## 3 – Topology

![Topology](https://raw.github.com/emersonbarea/RouteFlow-ExaBGP/master/RouteFlow-ExaBGP_Topology.png)

## 4 – lxc

There are 4 lxc routers, named rfvmA, rfvmB, rfvmC and rfvmD. B, C and D are running a Quagga router engine that area using BGP routing protocol to administer
