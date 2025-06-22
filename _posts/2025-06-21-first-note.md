---
title: First Note
author: mtai
---

{% include note-header.html %}
{% include dark-toggle.html %}

# Deploy vJunosEvolved on KVM

## Perquisite 
 - KVM
 - OpenVswitch

## Overview

vJunosEvolved is a single virtual machine (VM) that you can use only in labs and not in the production environment. The vJunosEvolved is built using PTX10001-36MR as a reference Juniper switch, which is a fixed-configuration packet transport router on the Junos® OS Evolved platform. The vJunosEvolved Routing Engine and the vBT-COSIM (a virtual BT chip) that performs the packet processing run on the same VM. Instead of using hardware switches, you can use the vJunosEvolved to start the Junos software for testing the network configurations and protocols. The vJunosEvolved virtual platform primarily acts as a test platform for lab simulations for the customers.

## vJunosEvolve Architecture

## Download the software

## KVM VM XML file example

```xml
<domain type='kvm' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>
  <name>vjunos01</name>
  <memory unit='KiB'>8388608</memory>
  <currentMemory unit='KiB'>8388608</currentMemory>
  <vcpu placement='static'>4</vcpu>
  <resource>
    <partition>/machine</partition>
  </resource>
  <os>
    <type arch='x86_64' machine='pc-i440fx-rhel7.6.0'>hvm</type>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <pae/>
  </features>
  <clock offset='utc'/>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <devices>
    <emulator>/usr/libexec/qemu-kvm</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2' cache='writeback'/>
      <source file='/var/lib/libvirt/images/vjunos01/vJunosEvolved-23.2R1-S1.8-EVO.qcow2'/>
      <backingStore/>
      <target dev='vda' bus='virtio'/>
      <alias name='virtio-disk0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </disk>
    <controller type='pci' index='0' model='pci-root'>
      <alias name='pci.0'/>
    </controller>
    <controller type='usb' index='0' model='piix3-uhci'>
      <alias name='usb'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x2'/>
    </controller>
    <interface type='bridge'>
      <source bridge='virbr1'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>
    <interface type='bridge'>
      <source bridge='ovsEXTbr'/>
      <virtualport type='openvswitch'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </interface>
    <serial type='pty'>
      <source path='/dev/pts/1'/>
      <target type='isa-serial' port='0'>
        <model name='isa-serial'/>
      </target>
      <alias name='serial0'/>
    </serial>
    <console type='pty' tty='/dev/pts/1'>
      <source path='/dev/pts/1'/>
      <target type='serial' port='0'/>
      <alias name='serial0'/>
    </console>
    <memballoon model='virtio'>
      <alias name='balloon0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
    </memballoon>
  </devices>
  <seclabel type='dynamic' model='dac' relabel='yes'>
    <label>+107:+107</label>
    <imagelabel>+107:+107</imagelabel>
  </seclabel>
  <qemu:commandline>
    <qemu:arg value='-smbios'/>
    <qemu:arg value='type=0,vendor=Bochs,version=Bochs'/>
    <qemu:arg value='-smbios'/>
    <qemu:arg value='type=3,manufacturer=Bochs'/>
    <qemu:arg value='-smbios'/>
    <qemu:arg value='type=1,manufacturer=Bochs,product=Bochs,serial=chassis_no=0:slot=0:type=1:assembly_id=0x0D20:platform=251:master=0:channelized=no'/>
  </qemu:commandline>
</domain>
```

## Start the VM

```
# virsh start vjunos01 --console
Domain vjunos01 started
Connected to domain vjunos01
Escape character is ^]
Watchdog set to 500 seconds
Installing/Mounting on disk /dev/vda mapped to device virtio0
Coming back from an uncontrolled reboot, will recreate /data!
Processing /dev/vda2 for mount on /soft ...[checking]..[ok 1]..[mounting]..done
Processing /dev/vda7 for mount on /var ...[checking]..[ok 1]..[mounting]..done
Processing /dev/vda5 for mount on /data ...[checking]..[strong check]..[saving]..[rebuilding]..[mounting]..[restoring]..done
Processing /dev/vda6 for mount on /data/config ...[checking]..[ok 0]..[mounting]..done
Processing /data/var/opt_fs for mount on /data/var/external ...[checking]..[ok 1]..[mounting]..done
mkswap: /dev/vda3: warning: wiping old swap signature.
Setting up swapspace version 1, size = 4 GiB (4294963200 bytes)
no label, UUID=cbd70fda-f508-47b0-896f-5ee6ab89e22a
Processing /dev/vda1 for mount on /boot ...[checking]..[ok 1]..[mounting]..done
Done with local filesystems setup.
Postinstall in progress...App manifest not defined in capdb. Skip customizing app policies for this platform ()
done
Installing kexec kernel...done
Warning - Empty root password found
*** Error - Config file not found !
warning: No configured root password
[   36.057972] ip_local_port_range: prefer different parity for start/end values.

Juniper Linux Distribution 3.0.2 re0 ttyS0

re0 login:
```

**Use username root to login, then issue cli**

```
re0 login: root
--- JUNOS 23.2R1-S1.8-EVO Linux (none) 5.2.60-yocto-standard-g12d8464 #1 SMP PREEMPT Sun May 21 01:02:49 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
[vrf:none] root@re0:~# cli
root@re0>
```

> Note: It takes few minutes for vfpc to come online

## vJunos base configuration example

```
set version 23.2R1-S1.8-EVO
set groups global interfaces lo0 unit 0 family inet address 127.0.0.1/24
set system host-name vjunosbase
set system root-authentication encrypted-password "$6$WmPIhBSM$G5oayCZgeYCi0.aXfoJXqwu64sLzAgp1/8KcnA8mSTUaR3u1S99zxrqGkYfBKvlhjCqrR2PRjlfY1VQO7wX9x1"
set system scripts op allow-url-for-python
set system scripts language python
set system login user admin uid 2000
set system login user admin class super-user
set system login user admin authentication encrypted-password "$6$/pY6DXZS$fy.ZaQeIHrtTnp8v/V.vcCdBIlX8sSZZx8OavxlYmSvnXi6BCbbNsDgUV0ofgkseE9QURGHwwP89QwBrse82d0"
set system login user admin authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC1pp46pzwLjqSwugiTqk7+Hc6KROa59tu12gHDLLCiYox1y/hsE+/dNGXZMrQmmImG6EKmlwSE0433EltqmwVTGXRtx9WAn4rHzBNZ7yL9w7jzYZui08eFVZxoheLJYUY11KBEnqOGsWzSNFIwoL0BHoiaolvqQongPT4g8rM3Pq+Z4QEPgH/PIcwVo8bfKOng3wD2YHFFCZIQ5eqFgHBtLobprJaDPhei2M0kAaIxpfIl0xvh9BnmoYVxSq6i33UWD+XnpY4Plggvh82UUqprprNZgJyroUpoG/rILeZ/DeVK/8kmYCQxtX4wR+1BHReGNVsL93VMj9ZenKCNJ75p root@azkvmhost01.softlayer-internal-network-labs.cloud"
set system syslog user * any emergency
set system syslog file interactive-commands interactive-commands any
set system syslog file messages any notice
set system syslog file messages authorization info
set system services ssh root-login allow
set system services ssh protocol-version v2
set system services netconf ssh port 830
set system time-zone UTC
set system management-instance
set system name-server 8.8.8.8
set system name-server 8.8.4.4
set system processes routing force-64-bit
set system processes nlsd enable
set system ntp server 206.210.192.99 prefer
set chassis aggregated-devices ethernet device-count 64
set interfaces et-0/0/0 ether-options 802.3ad ae1
set interfaces ae1 flexible-vlan-tagging
set interfaces ae1 mtu 9184
set interfaces ae1 encapsulation flexible-ethernet-services
set interfaces ae1 unit 4090 vlan-id 4090
set interfaces ae1 unit 4090 family inet address 169.254.254.200/24
set interfaces re0:mgmt-0 unit 0 family inet dhcp
set policy-options policy-statement CHASSIS-LACP-LOADBALANCE then load-balance per-packet
set routing-instances lab_mgmt instance-type virtual-router
set routing-instances lab_mgmt routing-options static route 0.0.0.0/0 next-hop 169.254.254.1
set routing-instances lab_mgmt interface ae1.4090
set routing-instances mgmt_junos routing-options static route 0.0.0.0/0 next-hop 192.168.88.1
set routing-options forwarding-table export CHASSIS-LACP-LOADBALANCE
set protocols bgp mtu-discovery
set protocols bgp log-updown
set protocols ldp track-igp-metric
set protocols ldp deaggregate
set protocols mpls path-mtu rsvp mtu-signaling
set protocols mpls traffic-engineering mpls-forwarding
set protocols mpls optimize-aggressive
set protocols mpls smart-optimize-timer 180
set protocols mpls optimize-switchover-delay 60
set protocols mpls ipv6-tunneling
set protocols ospf traffic-engineering shortcuts
set protocols lldp port-id-subtype interface-name
set protocols lldp interface all
```
