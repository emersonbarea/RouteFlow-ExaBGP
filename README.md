# Welcome

This version of [RouteFlow](http://cpqd.github.io/RouteFlow/) is responsible for demonstrate the behavior of a topology dynamically routed via BGP, where the [Quagga](http://www.nongnu.org/quagga/) engine is used with [ExaBGP](https://code.google.com/p/exabgp/) to route the topology used in this example.

# Overview

To create the proposed environment, besides RouteFlow, Quagga and ExaBGP, it is also required to use [Mininet](http://mininet.org/). The example of topology used in this case was [Four routers running OSPF] (https://sites.google.com/site/routeflow/documents/tutorial2-four-routers-with-ospf), where there was only a change in the rfvmA virtual machine, taking off Quagga and using ExaBGP instead.

# Building

First of all, you need have RouteFlow built and working. RouteFlow can be taken [here](https://github.com/CPqD/RouteFlow/blob/master/README.md#building). After that, you need clone this repository `git clone https://github.com/emersonbarea/RouteFlow-ExaBGP.git` and use the same Mininet scripts like in RouteFlow [rftest2](https://sites.google.com/site/routeflow/documents/tutorial2-four-routers-with-ospf).

# Environment

## 1 – createExaBGP

This script creates lxc routers using `config/ExaBGP` configurations files. RfvmA configuration file creates a lxc that includes `/etc/init.d/exabgp` script that is responsible for start and stop ExaBGP engine.

## 2 – rftestExaBGP

This script starts RouteFlow services.

After created and started RouteFlow services, Mininet should be used to mount the environment for testing. To verify the correct operation, you just have to examine the routing table of each lxc machine (rfvmA, rfvmB, rfvmC e rfvmD). Note that the route announced by ExaBGP and Quagga are different in rfvmA. For example:

Quagga: publishes 172.31.1.0/24

rfvmA example BGP Quagga code: [here](https://github.com/emersonbarea/RouteFlow-ExaBGP/blob/master/config/ExaBGP/rfvmA/rootfs/etc/quagga/bgpd.conf)

```
password routeflow
enable password routeflow
!
router bgp 65001
        bgp router-id 172.31.1.1
        network 172.31.1.0 mask 255.255.255.0
        neighbor 10.0.0.2 remote-as 65002
        neighbor 30.0.0.3 remote-as 65003
        neighbor 50.0.0.4 remote-as 65004
        no auto-summary
!
log file /var/log/quagga/bgpd.log
```

ExaBGP: publishes 172.31.1.100/32



## 3 – Topology

![Topology](/RouteFlow-ExaBGP_Topology.png)


## 4 – lxc

There are four lxc routers, named **rfvmA**, **rfvmB**, **rfvmC** and **rfvmD**. RfvmB, C and D are running a Quagga router engine that uses BGP routing protocol to mount all route tables. RfvmA is running ExaBGP engine that is responsible to inject routes in B, C and D. RfvmA configuration file can be accessed in lxc rfvmA router, under `/exabgp-3.1.10/etc/exabgp/exabgp.conf`, but changes can be done at rfvmA configuration before running createExaBGP script.

RfvmA has a script `/etc/init.d/exabgp` that is responsible for start and stop ExaBGP service. This script configures correct IP addresses in lxc rfvmA interfaces, starts ExaBGP service using correct configuration file and puts statics routes to back communication to rfvmB, C and D. When it is used with **stop** parameter, all ExaBGP configuration is taked off.
