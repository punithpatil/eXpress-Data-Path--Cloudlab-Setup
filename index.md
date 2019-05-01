## CloudLab Setup and Configurations 

The folowing containes the specifc details to our CloudLab setup. 

#### XDP supporting NIC drivers

An approximate list of drivers supporting XDP programs was found in the [iovisor/bcc](https://github.com/iovisor/bcc/blob/master/docs/kernel-versions.md#xdp) project. We are interested in the hardware spec for [CloudLab Utah](http://docs.cloudlab.us/hardware.html). The `xl170` nodes have Two Dual-port Mellanox ConnectX-4 25 GB NIC (PCIe v3.0, 8 lanes). This matches the suppriting driver Mellanox `mlx4` driver.

Provisioning baremetal nodes with the `xl170` as the Hardware Type in CloudLab Utah servers provies the support to run XDP programs.

Note: In CloudLab only CloudLab Utah servers support the `xl170` Hardware Type. The only available NICs that support XDP at the time are Mellanox ConnectX-4.

To verify that the correct NIC exists check the PCIe connected hardware. `xl170` nodes provide 2 Mellanox NICs with 2 interfaces each. They will show up as 4 Ethernet controllers.

```console
variable@xdp-node:~$ lspci -v | grep -A 5 Mellanox
03:00.0 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
	Subsystem: Hewlett Packard Enterprise MT27710 Family [ConnectX-4 Lx]
	Physical Slot: 1
	Flags: bus master, fast devsel, latency 0, IRQ 16, NUMA node 0
	Memory at 94000000 (64-bit, prefetchable) [size=32M]
	Capabilities: <access denied>
--
03:00.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
	Subsystem: Hewlett Packard Enterprise MT27710 Family [ConnectX-4 Lx]
	Physical Slot: 1
	Flags: bus master, fast devsel, latency 0, IRQ 17, NUMA node 0
	Memory at 96000000 (64-bit, prefetchable) [size=32M]
	Capabilities: <access denied>
--
07:00.0 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
	Subsystem: Hewlett Packard Enterprise MT27710 Family [ConnectX-4 Lx]
	Flags: bus master, fast devsel, latency 0, IRQ 16, NUMA node 0
	Memory at 98000000 (64-bit, prefetchable) [size=32M]
	Capabilities: <access denied>
	Kernel driver in use: mlx5_core
--
07:00.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
	Subsystem: Hewlett Packard Enterprise MT27710 Family [ConnectX-4 Lx]
	Flags: bus master, fast devsel, latency 0, IRQ 17, NUMA node 0
	Memory at 9a000000 (64-bit, prefetchable) [size=32M]
	Capabilities: <access denied>
	Kernel driver in use: mlx5_core
```
#### CloudLab Profile

The experiments were set up with one baremetal node of type `xl170` and another baremetal node with no specifc type selection. There is a connection provied between the two baremetal nodes over a link of type lan and no specifics provided for vLan configurations. 

Here is the XML of the basic CloudLab profile used.

```xml
<rspec xmlns="http://www.geni.net/resources/rspec/3" xmlns:emulab="http://www.protogeni.net/resources/rspec/ext/emulab/1" xmlns:jacks="http://www.protogeni.net/resources/rspec/ext/jacks/1" xmlns:tour="http://www.protogeni.net/resources/rspec/ext/apt-tour/1" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.geni.net/resources/rspec/3    http://www.geni.net/resources/rspec/3/request.xsd" type="request">
   <node client_id="xdp-node">
      <icon xmlns="http://www.protogeni.net/resources/rspec/ext/jacks/1" url="https://www.emulab.net/protogeni/jacks-stable/images/server.svg" />
      <sliver_type name="raw-pc">
         <disk_image name="urn:publicid:IDN+utah.cloudlab.us+image+cu-bison-lab-PG0//xdp-headers-ubuntu18" />
      </sliver_type>
      <hardware_type name="xl170" />
      <services />
      <interface client_id="interface-0" />
   </node>
   <node client_id="traffic-node">
      <icon xmlns="http://www.protogeni.net/resources/rspec/ext/jacks/1" url="https://www.emulab.net/protogeni/jacks-stable/images/server.svg" />
      <sliver_type name="raw-pc" />
      <services />
      <interface client_id="interface-1" />
   </node>
   <link client_id="link-0">
      <link_type name="lan" />
      <interswitch xmlns="http://www.protogeni.net/resources/rspec/ext/emulab/1" allow="no" />
      <interface_ref client_id="interface-0" />
      <interface_ref client_id="interface-1" />
      <site xmlns="http://www.protogeni.net/resources/rspec/ext/jacks/1" id="undefined" />
   </link>
   <rspec_tour xmlns="http://www.protogeni.net/resources/rspec/ext/apt-tour/1">
      <description xmlns="" type="markdown">Basic xdp experiment profile

xdp-node -&gt; has xdp headers
traffic-node -&gt; should be used to generate traffic. Standard Ubuntu-18 image
link -&gt; Only selected option 'force non trivial' in order to avoid using loopback interface</description>
   </rspec_tour>
</rspec>
```

#### TODO check list for CloudLab specific set up

- [x] XDP NIC support
- [x] CloudLab `xl170` Profile support
- [ ] disk space needed for kernel files
- [ ] document shared workspace and problems
- [ ] configurations for creating data set in CloudLab as a solution to shared work space issue
- [ ] image creation for new compiled kernel 
