# Model-driven network service abstraction for networking Linux containers on small PCs

![NLAN architecture](https://docs.google.com/drawings/d/1VauRM6d2A03gIPxFbaVYdZpP8Yadre8KqL53XnntqDI/pub?w=600&h=400)

This project unifies outputs from my two other projects "[neutron-lan](https://github.com/araobp/neutron-lan)" and "[tega](https://github.com/araobp/tega)".

##Background and motivation
- OpenDaylight MD-SAL is too heavy for networking Linux containers on my Raspberry Pi.
- YANG is incompatible with Python dict, Golang map and so on: I just want JSON-centric MD-SAL.
- As my hobby, I design a model-driven/event-driven architecture for networking Linux containers.

##NLAN services
- PTN: Packet Transport Network (Layer 1 and Layer 2)
- DVR: Distributed Virtual Switch and Distributed Virtual Router (Layer 2 and Layer 3)
- vHosts: netns-based virtual hosts
- Router: Quagga configuration

To be added:
- Links: direct linking(tun/tap)
- Bridges: non-distributed virtual switch
- VRF: netns-based VRF
- container-macvlan direct linking (skipping docker0)
- [netlink](https://github.com/milosgajdos83/tenus) in addition to "ip" and "brctl"

##Target use cases

Use case 1 has already been implemented, and use case 2 is being planned at the moment.

### Use case 1: Network simulation

This use case makes use of NLAN's PTN, vHosts and Router services.
![WAN simulation](https://docs.google.com/drawings/d/1VKfKlwnzWQ2-ImfXeB5uNegGBK0BnaGU_4lS8h4Qpcw/pub?w=640&h=480)

####Declarative state representations:
- [ptn-bgp.yaml](./etc/ptn-bgp.yaml)
- [ptn-ospf.yaml](./etc/ptn-ospf.yaml)

####Running the simulated network on Raspberry Pi
This is sort of micro NFV(Network Function Virtualization) on a single Rapsberry Pi.
- Nine virtual routers (Linux containers)
- Sixteen virutal hosts (netns)

You can learn how routing protocols work on this simulated network.

[Setting up the software on Raspberry Pi](./doc/RPI.md)

Log in the virtual routers with ssh, and try "ip" or "vtysh" commands:
- ip route
- ip addr
- ip link
- ip netns
- vtysh: show run
- vtysh: show ip route
- vtysh: show ip bgp
     :

####Quagga and GoBGP:
This use case makes use of Quagga, but [gobgp](https://github.com/osrg/gobgp) may optionally be used as Route Reflector or Route Server on "RR" container in the fig above.
- [gobgpd.conf](./etc/gobgpd.conf)

You can also launch gobgpd from NLAN agent by including "EmbeddedBgp: true" in your NLAN state file:
```
      Router:
        Loopback: 10.1.1.5/32
        EmbeddedBgp: true
        Bgp:
          - As: 100
            Neighbors:
              - Peer: 10.200.1.101
                RemoteAs: 100
                RouteReflectorClient: true
              - Peer: 10.200.1.102
                RemoteAs: 100
                RouteReflectorClient: true
              - Peer: 10.200.1.103
                RemoteAs: 100
                RouteReflectorClient: true
              - Peer: 10.200.1.104
                RemoteAs: 100
                RouteReflectorClient: true
```
- [Using gobgp command](./doc/GOBGP.md)

### Use case 2: SOHO NFV (Network Functions Virtualization)

This is the next use case I am going to work on... (as my hobby: not so practical)

![SONO-NFV](https://docs.google.com/drawings/d/11fJUimZVrGxqAdq-hJK4abDu0ZThkfHGtbl_94zW0rQ/pub?w=640&h=480)

##Network simulation with Linux containers
I use Linux containers as virtual routers, and this tool will set up virtual links (L0/L1) and virtual switches (L2) over the containers. Then I will run Quagga/Zebra(L3) daemons over the virtual routers to study how legacy routing protocols work.
- [An example of such a network](https://camo.githubusercontent.com/3f15c9634b2491185ec680fa5bb7d19f6f01146b/68747470733a2f2f646f63732e676f6f676c652e636f6d2f64726177696e67732f642f31564b664b6c776e7a5751322d496d6658654235754e656747424b30426e6147555f346c53386834517063772f7075623f773d39363026683d373230)
- [Working with Docker for network simulation](https://camo.githubusercontent.com/77cf473ea9499432e57b06a951f5f5248419f9e1/68747470733a2f2f646f63732e676f6f676c652e636f6d2f64726177696e67732f642f313631426e383077384a5a4b513742586d496f306272377851346b71456442635f585a3235347a754f5253552f7075623f773d36383026683d343030)

##Key technologies used in this project
- Open vSwitch, OVSDB/JSON-RPC([RFC7047](https://tools.ietf.org/html/rfc7047)) and [vxlan](https://tools.ietf.org/html/rfc7348)
- [docker](https://github.com/docker/docker)
- JSON/YAML
- Protocol buffers
- quagga and [gobgp](https://github.com/osrg/gobgp)

##Interesting
- [Cumulus Linux](https://cumulusnetworks.com/)
- [OpenSwitch] (http://www.openswitch.net/)
- [socketplane](https://github.com/socketplane/socketplane)
- [goplane](https://github.com/osrg/goplane)
- [bgp sdn](https://tools.ietf.org/html/draft-lapukhov-bgp-sdn-00)
- [nsq](https://github.com/nsqio/nsq)

#NLAN installation

[Step 1] Make a Docker image named "router" following the instruction [here](./docker/SETUP.md).

[Step 2] Install and start tega db:
```
$ pip3 install mako
$ go get github.com/araobp/tega/driver
```

For Hypriot/RaspberryPi, you need to export this environment variable:
```
$ export SETUP_SCRIPT=setup_rpi.sh
```
For Debian/Ubuntu, you do not need to export the variable above. 

Then start tega db:
```
$ cd scripts
$ ./tegadb &
Namespace(datadir='./var', extensions='/root/work/src/github.com/araobp/nlan/plugins/nlan', mhost=None, mport=None, port=8888, sync=False, syncpath=None, tegaid='global')

   __
  / /____  ____ _____ _
 / __/ _ \/ __ `/ __ `/
/ /_/  __/ /_/ / /_/ /
\__/\___/\__, /\__,_/
        /____/
tega_id: global, sync: server

INFO:2016-01-29 14:31:21,319:Reloading log from ./var...
INFO:2016-01-29 14:31:21,844:Reloading done
INFO:2016-01-29 14:31:24,216:plugin attached to idb: Deployment
INFO:2016-01-29 14:31:24,220:plugin attached to idb: Topo
INFO:2016-01-29 14:31:24,282:plugin attached to idb: IpAddressManagement
INFO:2016-01-29 14:31:24,342:plugin attached to idb: Template
```

[Step 3] Execute nlan.ipam (IP address management) function on tega db to generate (secondary) IP addresses of each containers:
```
$ cd scripts
$ ./cli
[tega: 1] nlan.ipam('10.10.10.1','pe1','pe2','pe3','pe4','rr','ce1','ce2','ce3','ce4')
[tega: 2] get nlan.ip
{ce1: 10.10.10.6/2, ce2: 10.10.10.7/2, ce3: 10.10.10.8/2, ce4: 10.10.10.9/2, pe1: 10.10.10.1/24,
  pe2: 10.10.10.2/24, pe3: 10.10.10.3/24, pe4: 10.10.10.4/24, rr: 10.10.10.5/2}
```
[Step 3]
Try this at the tega CLI to put "ptn-bgp" state onto tega db: 
```
[tega: 3] nlan.template(filename='ptn-bgp.yaml')
```
The script sets up [this network](https://camo.githubusercontent.com/3f15c9634b2491185ec680fa5bb7d19f6f01146b/68747470733a2f2f646f63732e676f6f676c652e636f6d2f64726177696e67732f642f31564b664b6c776e7a5751322d496d6658654235754e656747424b30426e6147555f346c53386834517063772f7075623f773d39363026683d373230).

[Step 4(option)]
You may take a snapshop of tega db to make tega db's start-up faster:
```
[tega: 4] ss 
```

[Step 5] Execute the following command to build Docker image with NLAN agent embedded and to start the containers:

```
[tega: 5] nlan.deploy() 
```

NLAN agent on each container connects to tega db to fetch NLAN state.

[Step 6] Confirm that all the containers are running:
```
[tega: 6] subscribers
Deployment: [Deployment]
IpAddressManagement: [IpAddressManagement]
Template: [Template]
Topo: [Topo, nlan.state]
ce1: [nlan.raw_request, nlan.raw_request.ce1]
ce2: [nlan.raw_request, nlan.raw_request.ce2]
ce3: [nlan.raw_request, nlan.raw_request.ce3]
ce4: [nlan.raw_request, nlan.raw_request.ce4]
pe1: [nlan.raw_request, nlan.raw_request.pe1]
pe2: [nlan.raw_request, nlan.raw_request.pe2]
pe3: [nlan.raw_request, nlan.raw_request.pe3]
pe4: [nlan.raw_request, nlan.raw_request.pe4]
rr: [nlan.raw_request, nlan.raw_request.rr]
```

[Step 7] Open ssh session to the containers:
```
$ cd scripts 
$ ./ssh.sh pe1
       :
$ ./ssh.sh ce1
       :
```
The password is "root".

Then you can do whatever you try.

#Development environment setup

##Building Golang and protobuf for 32bit Linux
I use a very old PC with a 32bit CPU at home, so I need to build 32bit binary from source codes:
- Go lang installation: https://golang.org/dl/
- Protobuf build and installation: https://github.com/google/protobuf/blob/master/INSTALL.txt
```
$ autoconf
$ ./autogen.sh
$ ./configure
$ make
$ make install
```
- Add /usr/local/lib to LD_LIBRARY_PATH
```
$ export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARLY_PATH

```

##Go plugin for vim

Install [vim-go](https://github.com/fatih/vim-go) to your vim.
