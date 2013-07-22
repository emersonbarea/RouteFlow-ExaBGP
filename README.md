# Welcome

This version of [RouteFlow](http://cpqd.github.io/RouteFlow/) is responsible for demonstrate the behavior of a topology dynamically routed via BGP, where the [Quagga](http://www.nongnu.org/quagga/) engine is used with [ExaBGP](https://code.google.com/p/exabgp/) to route the topology used in this example.

# Overview

To create the proposed environment, besides RouteFlow, Quagga and ExaBGP, it is also required to use [Mininet](http://mininet.org/). The example of topology used in this case was [Four routers running OSPF] (https://sites.google.com/site/routeflow/documents/tutorial2-four-routers-with-ospf) with some changes. Firstl, Quagga OSPF configuration was removed and BGP was configured to be the routing protocol used. Second, `rfvmA` has Quagga disabled and ExaBGP was configured to inject BGP routes in `rfvmB`, `rfvmC` and `rfvmD`.

# Building

First of all, you need have RouteFlow built and working. RouteFlow can be taken [here](https://github.com/CPqD/RouteFlow/blob/master/README.md#building). After that, you need clone this repository `git clone https://github.com/emersonbarea/RouteFlow-ExaBGP.git` and use the same Mininet scripts like in RouteFlow [rftest2](https://sites.google.com/site/routeflow/documents/tutorial2-four-routers-with-ospf).

# Environment

## 1 – Topology

![Topology](/RouteFlow-ExaBGP_Topology.png)

## 2 – lxc

There are four lxc routers, named `rfvmA`, `rfvmB`, `rfvmC` and `rfvmD`. RfvmB, C and D are running a Quagga router engine that uses BGP routing protocol (in EBGP mode) to mount all route tables. RfvmA is running ExaBGP engine that is responsible to inject routes in rfvmB, C and D. RfvmA configuration file can be accessed in lxc rfvmA router, under `/exabgp-3.1.10/etc/exabgp/exabgp.conf`, but changes can be done at rfvmA configuration before running `createExaBGP` script.

RfvmA has a script `/etc/init.d/exabgp` that is responsible for start and stop ExaBGP service. This script configures correct IP addresses in lxc rfvmA interfaces, starts ExaBGP service using correct configuration file and puts statics routes to back communication to rfvmB, C and D. When it is used with **stop** parameter, all ExaBGP configuration is taked off.

## 3 – Script createExaBGP

This [script](https://github.com/emersonbarea/RouteFlow-ExaBGP/blob/master/createExaBGP) creates lxc routers using `config/ExaBGP` configuration files. All lxc routers has same softwares included in [Four routers running OSPF] (https://sites.google.com/site/routeflow/documents/tutorial2-four-routers-with-ospf), plus ExaBGP included with `wget http://exabgp.googlecode.com/files/exabgp-3.1.10.tgz` command.

### 3.1 - `RfvmA` configuration files

`CreateExaBGP` script includes `rfvmA` scripts creation like `/etc/init.d/exabgp`, that is responsible for start and stop ExaBGP engine. Some others importants functions of `/etc/init.d/exabgp` script are:

1 - Configure `rfvmA` IP addresses

```
ifconfig eth1 172.31.1.1/24
ifconfig eth2 10.0.0.1/24
ifconfig eth3 30.0.0.1/24
ifconfig eth4 50.0.0.1/24
```
This configuration was previously done by Quagga.

2 - Configure back routing

ExaBGP is not capable of receive BGP routes from other Quagga lxc, so is necessary to create back routes to `rfvmB`, `rfvmC` and `rfvmD`.

```
route add -net 172.31.2.0/24 gw 10.0.0.2
route add -net 172.31.3.0/24 gw 30.0.0.3
route add -net 172.31.4.0/24 gw 50.0.0.4
```

It is possible use other routing methods, but this is simplest for this scenario.

## 4 – Script rftestExaBGP

This [script](https://github.com/emersonbarea/RouteFlow-ExaBGP/blob/master/rftestExaBGP) starts all RouteFlow services. `RfvmA` is started running ExaBGP engine and `rfvmB`, `rfvmC` and `rfvmD` are started running Quagga engine.

After created and started RouteFlow services, Mininet should be used to mount the environment for testing. To verify the correct operation, you just have to examine the routing table of each lxc machine (rfvmA, rfvmB, rfvmC e rfvmD). Note that the route announced by ExaBGP and Quagga are different in rfvmA. For example:

Quagga: publishes 172.31.1.0/24

rfvmA BGP Quagga code:

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

[Quagga code file](https://github.com/emersonbarea/RouteFlow-ExaBGP/blob/master/config/ExaBGP/rfvmA/rootfs/etc/quagga/bgpd.conf)


ExaBGP: publishes 172.31.1.100/32

rfvmA example BGP ExaBGP code: 

```
neighbor 30.0.0.3{
        description "a quagga test peer";
        router-id 30.0.0.1;
        local-address 30.0.0.1;
        local-as 65001;
        peer-as 65003;
        static {
                route 172.31.1.100/32 next-hop 30.0.0.1;
        }
}
...
```

[ExaBGP code file] (https://github.com/emersonbarea/RouteFlow-ExaBGP/blob/master/config/ExaBGP/rfvmA/rootfs/exabgp-3.1.10/etc/exabgp/exabgp.conf)

There not  a technical motivation for the difference between ExaBGP and Quagga routes, just for facilitate the visualization of when using one or other.

## 5 – Results

1 - `RfvmA` routing table when it is running Quagga and ExaBGP

`RfvmA` Quagga routing table when it is running **Quagga**

```
root@rfvmA:/home/ubuntu# vtysh
rfvmA# show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP, O - OSPF,
       I - ISIS, B - BGP, > - selected route, * - FIB route
B   10.0.0.0/24 [20/1] via 10.0.0.2 inactive, 00:12:43
C>* 10.0.0.0/24 is directly connected, eth2
C>* 30.0.0.0/24 is directly connected, eth3
B>* 40.0.0.0/24 [20/1] via 10.0.0.2, eth2, 00:12:43
C>* 50.0.0.0/24 is directly connected, eth4
C>* 127.0.0.0/8 is directly connected, lo
C>* 172.31.1.0/24 is directly connected, eth1
B>* 172.31.2.0/24 [20/0] via 10.0.0.2, eth2, 00:12:43
B>* 172.31.3.0/24 [20/0] via 30.0.0.3, eth3, 00:12:46
B>* 172.31.4.0/24 [20/0] via 50.0.0.4, eth4, 00:12:46
B   192.169.1.0/24 [20/1] via 10.0.0.2, eth2, 00:12:43
C>* 192.169.1.0/24 is directly connected, eth0
```

**Linux** routing table when `rfvmA` is running ExaBGP

```
root@rfvmA:/home/ubuntu# route -n
Kernel IP routing table
Destination    Gateway    Genmask          Flags     Metric    Ref    Use    Iface
10.0.0.0       0.0.0.0    255.255.255.0    U         0         0      0      eth2
30.0.0.0       0.0.0.0    255.255.255.0    U         0         0      0      eth3
50.0.0.0       0.0.0.0    255.255.255.0    U         0         0      0      eth4
172.31.1.0     0.0.0.0    255.255.255.0    U         0         0      0      eth1
172.31.2.0     10.0.0.2   255.255.255.0    UG        0         0      0      eth2
172.31.3.0     30.0.0.3   255.255.255.0    UG        0         0      0      eth3
172.31.4.0     50.0.0.4   255.255.255.0    UG        0         0      0      eth4
192.169.1.0    0.0.0.0    255.255.255.0    U         0         0      0      eth0
root@rfvmA:/home/ubuntu#
```

2 - `RfvmB` routing table when `rfvmA` is running Quagga and ExaBGP

`RfvmB` Quagga routing table when `rfvmA` is running **Quagga**

```
root@rfvmB:/home/ubuntu# vtysh
rfvmB# show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP, O - OSPF,
       I - ISIS, B - BGP, > - selected route, * - FIB route
C>* 10.0.0.0/24 is directly connected, eth2
C>* 40.0.0.0/24 is directly connected, eth3
C>* 127.0.0.0/8 is directly connected, lo
B>* 172.31.1.0/24 [20/0] via 10.0.0.1, eth2, 00:00:02
C>* 172.31.2.0/24 is directly connected, eth1
B>* 172.31.3.0/24 [20/0] via 40.0.0.4, eth3, 00:05:02
B>* 172.31.4.0/24 [20/0] via 40.0.0.4, eth3, 00:05:32
C>* 192.169.1.0/24 is directly connected, eth0
```

`RfvmB` Quagga routing table when `rfvmA` is running **ExaBGP**

```
root@rfvmB:/home/ubuntu# vtysh
rfvmB# show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP, O - OSPF,
       I - ISIS, B - BGP, > - selected route, * - FIB route
C>* 10.0.0.0/24 is directly connected, eth2
C>* 40.0.0.0/24 is directly connected, eth3
C>* 127.0.0.0/8 is directly connected, lo
B>* 172.31.1.100/32 [20/0] via 10.0.0.1, eth2, 00:04:19
C>* 172.31.2.0/24 is directly connected, eth1
B>* 172.31.3.0/24 [20/0] via 40.0.0.4, eth3, 00:02:10
B>* 172.31.4.0/24 [20/0] via 40.0.0.4, eth3, 00:02:40
C>* 192.169.1.0/24 is directly connected, eth0
```

3 - Mininet `pingall` output when `rfvmA` is running ExaBGP

```
mininet> pingall
*** Ping: testing ping reachability
h1 -> h2 h3 h4
h2 -> h1 h3 h4
h3 -> h1 h2 h4
h4 -> h1 h2 h3
*** Results: 0% dropped (0/12 lost)
mininet>
```
