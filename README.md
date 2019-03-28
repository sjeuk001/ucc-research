# ucc-research
The following information are considered a base to a setup that can be used to further develop UCC and tenant-specific entity identification in cloud. UCC is a PhD research proposal done at University College London. To learn more about UCC and the research done please use the following link: https://ieeexplore.ieee.org/document/6846532

The following figure depicts an example topology that can be build using the information provided. The configuration information are necessary for a basic setup that enable a data centre like environment. VPP instances are running within a docker environment, are interconnected using memif interfaces and use DPDK interfaces to connect to the outside world (i.e. to access traffic generator). 

![alt text](https://github.com/sjeuk001/ucc-research/blob/master/vpp_test_network_implementation.jpg)
 

```ruby
VPP11:
                set interface ip address TenGigabitEthernet62/0/0 db11::1/64
                set interface state TenGigabitEthernet62/0/0 up
                create memif socket id 1 filename /memifs/VPP11-VPP12
                create interface memif id 1 socket-id 1 slave
                set interface ip address memif1/1 2000::1/64
                set interface state memif1/1 up
                ip route ::/0 via 2000::2
```   
```ruby
VPP-CORE:
                create memif socket id 1 filename /memifs/VPP12-VPPCORE
                create interface memif id 1 socket-id 1 master
                set interface ip address memif1/1 2000::2/64
                set interface state memif1/1 up
                create memif socket id 2 filename /memifs/VPP22-VPPCORE
                create interface memif id 2 socket-id 2 master
                set interface ip address memif2/2 2001::2/64
                set interface state memif2/2 up
                ip route db11::0/64 via 2000::1
                ip route db22::0/64 via 2001::1
```
```ruby
VPP21:
                set interface ip address TenGigabitEthernet68/0/0 db22::1/64
                set interface state TenGigabitEthernet68/0/0 up
                create memif socket id 2 filename /memifs/VPP21-VPP22
                create interface memif id 2 socket-id 2 slave
                set interface ip address memif2/2 2001::1/64
                set interface state memif2/2 up
                ip route ::/0 via 2001::2
```

L3 VPP Instances are started using the following commands: 

```ruby
VPP11:
vpp unix { log /tmp/vpp.log full-coredump cli-listen localhost:5002 } dpdk { socket-mem 1024,1024 uio-driver igb_uio dev 0000:62:00.0 {vlan-strip-offload off num-rx-queues 2}} cpu { main-core 2 corelist-workers 3-4 }
```
```ruby
VPP-CORE
vpp unix { log /tmp/vpp.log full-coredump cli-listen localhost:5002 } cpu { main-core 5 corelist-workers 6-7 }
```
```ruby
VPP21:
vpp unix { log /tmp/vpp.log full-coredump cli-listen localhost:5002 } dpdk { socket-mem 1024,1024 uio-driver igb_uio dev 0000:68:00.0 {vlan-strip-offload off num-rx-queues 2} } cpu { main-core 8 corelist-workers 9-10 }
```


The server is configured to optimize overall performance of the software based virtual switch VPP. The settings done here are typical to assure CPU cyles, memory access as well as numa node placements are defined properly to assure VPP does forward packets properly. 

![alt text](https://github.com/sjeuk001/ucc-research/blob/master/cpo_settings_ucs220m5.jpeg)

For further information on UCC or to collaborate on further development please reach out to: 
Sebastian Jeuk (ucabsej (at) ucl (dot) ac (dot) uk)
