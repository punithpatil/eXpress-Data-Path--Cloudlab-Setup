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

#### File System requirements

The nodes provisioned in CloudLab come with a file mounts that do not have enough space for building the kernel. There are two approaches that were tried.

1. Data set 
2. Shared file system for the entire project

##### Data Set

This is CloudLab's offering to suit needs where large training data has to be loaded for training models. It is offered as a NFS mount so we can mount it directly to the nodes and use it. This requires the change in the project profile to add the newly created data set. 

It is relativly simple to create a dataset in CloudLab, it is important to get a data set with a relatively long life and it must exist in the same CloudLab cluster as the project. Once the dataset is created we will use the URN provided to specify the mount in our project profile 

Example URN: `urn:publicid:IDN+utah.cloudlab.us:cu-bison-lab-pg0+stdataset+test`

`cu-bison-lab-pg0` is the project name

`test` is the name of the dataset

##### Experiment Profile Changes 

The basic CloudLab experimetn profile can be edited using the UI, to add the dataset it will be converted to a geni script. Here is an example with the support to add the data set in.

```python
"""Profile to perform kernel builds for BPF compatibility. Maintainer - punith.patil@colorado.edu"""

#
# NOTE: This code was machine converted. An actual human would not
#       write code like this!
#

# Import the Portal object.
import geni.portal as portal
# Import the ProtoGENI library.
import geni.rspec.pg as pg
# Import the Emulab specific extensions.
import geni.rspec.emulab as emulab

# Create a portal object,
pc = portal.Context()

# Create a Request object to start building the RSpec.
request = pc.makeRequestRSpec()

# Node node-0
node_0 = request.RawPC('node-0')
node_0.disk_image = 'urn:publicid:IDN+emulab.net+image+emulab-ops//UBUNTU18-64-STD'

# Dataset config that will be mounted to node-0
iface = node_0.addInterface()

fsnode = request.RemoteBlockstore("fsnode", "/dataset-path")
fsnode.dataset = "urn:publicid:IDN+utah.cloudlab.us:cu-bison-lab-pg0+stdataset+test"
fslink = request.Link("fslink")
fslink.addInterface(iface)
fslink.addInterface(fsnode.interface)
fslink.best_effort = True
fslink.vlan_tagging = True


# Print the generated rspec
pc.printRequestRSpec(request)
```

The nodes booted with this experiment as the setting will boot with the `test` dataset mounted on `/dataset-path`. This can be used as a scratch disk for the kernel build. 

##### Shared File System

Each node in CloudLab boots with a similar file system as shown below 

```console
variable@xdp-node:~$ df -h
Filesystem                                   Size  Used Avail Use% Mounted on
udev                                          32G     0   32G   0% /dev
tmpfs                                        6.3G  1.2M  6.3G   1% /run
/dev/sda1                                     16G  8.2G  6.8G  55% /
tmpfs                                         32G     0   32G   0% /dev/shm
tmpfs                                        5.0M     0  5.0M   0% /run/lock
tmpfs                                         32G     0   32G   0% /sys/fs/cgroup
ops.utah.cloudlab.us:/proj/cu-bison-lab-PG0  100G   30G   71G  30% /proj/cu-bison-lab-PG0
ops.utah.cloudlab.us:/share                   97G   16G   74G  18% /share
tmpfs                                        6.3G     0  6.3G   0% /run/user/20001
```

Without a dataset it is not possible to work on anything that uses more than 10GiB. The datasets on CloudLab can expire and it is painful to rebuild the kernel for every test. The approach we used was to use the shared file system provied for all nodes in the `cu-bison-lab-pg0` project

`ops.utah.cloudlab.us:/proj/cu-bison-lab-PG0  100G   30G   71G  30% /proj/cu-bison-lab-PG0`

The above file mount is 100GiB is size and is large enough for the kernel build. It will be mounted on all nodes that are created under the same project. This is persistant throughout the lifetime of the project itself.

###### Warning⚠️: Shared File System contains backup data for the project

It should be noted that the file `/proj/cu-bison-lab-PG0` also contains back up data of all experiments in the project. It is used by the IT Admins for backup related requests and system maintainability. If it is corrupted immediately contact the OPS team to restore it. Changing any of the files will disable the project and all creation of future nodes/experiments will fail.

#### TODO check list for CloudLab specific set up

- [x] XDP NIC support
- [x] CloudLab `xl170` Profile support
- [x] disk space needed for kernel files
- [x] document shared workspace and problems
- [x] configurations for creating data set in CloudLab as a solution to shared work space issue
- [ ] image creation for new compiled kernel 
