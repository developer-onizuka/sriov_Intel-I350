# sriov_Intel-I350

# 1. OS information
```
$ uname -r
5.4.0-89-generic

$ cat /etc/os-release 
NAME="Linux Mint"
VERSION="20.2 (Uma)"
ID=linuxmint
ID_LIKE=ubuntu
PRETTY_NAME="Linux Mint 20.2"
VERSION_ID="20.2"
HOME_URL="https://www.linuxmint.com/"
SUPPORT_URL="https://forums.linuxmint.com/"
BUG_REPORT_URL="http://linuxmint-troubleshooting-guide.readthedocs.io/en/latest/"
PRIVACY_POLICY_URL="https://www.linuxmint.com/"
VERSION_CODENAME=uma
UBUNTU_CODENAME=focal
```

# 2. Edit some config files
```
$ vi /etc/default/grub
...
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on vfio_iommu_type1.allow_unsafe_interrupts=1 iommu=pt pci=assign-busses"
...

$ sudo update-grub

$ sudo su
# cat <<EOF > /etc/modprobe.d/igb.conf 
options igb max_vfs=4
blacklist igbvf
EOF

# exit

$ sudo update-initramfs -u -k `uname -r`
$ reboot
```

# 3. Check
```
$ cat /sys/bus/pci/devices/0000\:06\:00.0/sriov_numvfs 
4
$ cat /sys/bus/pci/devices/0000\:06\:00.1/sriov_numvfs 
4

$ lspci |grep I350
06:00.0 Ethernet controller: Intel Corporation I350 Gigabit Network Connection (rev 01)
06:00.1 Ethernet controller: Intel Corporation I350 Gigabit Network Connection (rev 01)
06:10.0 Ethernet controller: Intel Corporation I350 Ethernet Controller Virtual Function (rev 01)
06:10.1 Ethernet controller: Intel Corporation I350 Ethernet Controller Virtual Function (rev 01)
06:10.4 Ethernet controller: Intel Corporation I350 Ethernet Controller Virtual Function (rev 01)
06:10.5 Ethernet controller: Intel Corporation I350 Ethernet Controller Virtual Function (rev 01)
06:11.0 Ethernet controller: Intel Corporation I350 Ethernet Controller Virtual Function (rev 01)
06:11.1 Ethernet controller: Intel Corporation I350 Ethernet Controller Virtual Function (rev 01)
06:11.4 Ethernet controller: Intel Corporation I350 Ethernet Controller Virtual Function (rev 01)
06:11.5 Ethernet controller: Intel Corporation I350 Ethernet Controller Virtual Function (rev 01)

$ sudo lspci -nnk -d 8086:1520
06:10.0 Ethernet controller [0200]: Intel Corporation I350 Ethernet Controller Virtual Function [8086:1520] (rev 01)
	Subsystem: Intel Corporation I350 Ethernet Controller Virtual Function [8086:00a2]
	Kernel modules: igbvf
06:10.1 Ethernet controller [0200]: Intel Corporation I350 Ethernet Controller Virtual Function [8086:1520] (rev 01)
	Subsystem: Intel Corporation I350 Ethernet Controller Virtual Function [8086:00a2]
	Kernel modules: igbvf
06:10.4 Ethernet controller [0200]: Intel Corporation I350 Ethernet Controller Virtual Function [8086:1520] (rev 01)
	Subsystem: Intel Corporation I350 Ethernet Controller Virtual Function [8086:00a2]
	Kernel modules: igbvf
06:10.5 Ethernet controller [0200]: Intel Corporation I350 Ethernet Controller Virtual Function [8086:1520] (rev 01)
	Subsystem: Intel Corporation I350 Ethernet Controller Virtual Function [8086:00a2]
	Kernel modules: igbvf
06:11.0 Ethernet controller [0200]: Intel Corporation I350 Ethernet Controller Virtual Function [8086:1520] (rev 01)
	Subsystem: Intel Corporation I350 Ethernet Controller Virtual Function [8086:00a2]
	Kernel modules: igbvf
06:11.1 Ethernet controller [0200]: Intel Corporation I350 Ethernet Controller Virtual Function [8086:1520] (rev 01)
	Subsystem: Intel Corporation I350 Ethernet Controller Virtual Function [8086:00a2]
	Kernel modules: igbvf
06:11.4 Ethernet controller [0200]: Intel Corporation I350 Ethernet Controller Virtual Function [8086:1520] (rev 01)
	Subsystem: Intel Corporation I350 Ethernet Controller Virtual Function [8086:00a2]
	Kernel modules: igbvf
06:11.5 Ethernet controller [0200]: Intel Corporation I350 Ethernet Controller Virtual Function [8086:1520] (rev 01)
	Subsystem: Intel Corporation I350 Ethernet Controller Virtual Function [8086:00a2]
	Kernel modules: igbvf
```

# 4. Attach VFs to guest OS
You might use the following Vagrantfile. This creates two Virtual Machine with one VF.
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"
#---------- sriov-0 ----------
  config.vm.define "sriov-0_192.168.33.113" do |server|
    server.vm.network "private_network", ip: "192.168.33.113"
    server.vm.hostname = "sriov-0"
    server.vm.provider "libvirt" do |kvm|
      kvm.memory = 8192 
      kvm.cpus = 2
      kvm.machine_type = "q35"
      kvm.cpu_mode = "host-passthrough"
      kvm.kvm_hidden = true
      kvm.pci :bus => '0x06', :slot => '0x10', :function => '0x0'
    end
    server.vm.provision "shell", inline: <<-SHELL
    SHELL
  end
#---------- sriov-1 ----------
  config.vm.define "sriov-1_192.168.33.114" do |server|
    server.vm.network "private_network", ip: "192.168.33.114"
    server.vm.hostname = "sriov-1"
    server.vm.provider "libvirt" do |kvm|
      kvm.memory = 8192 
      kvm.cpus = 2
      kvm.machine_type = "q35"
      kvm.cpu_mode = "host-passthrough"
      kvm.kvm_hidden = true
      kvm.pci :bus => '0x06', :slot => '0x10', :function => '0x4'
    end
    server.vm.provision "shell", inline: <<-SHELL
    SHELL
  end
end
```
After booting it, You can find the status on the Host Machine as like below:
```
$ ip link show enp6s0f0
2: enp6s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether a0:36:9f:a4:66:4a brd ff:ff:ff:ff:ff:ff
    vf 0     link/ether 0a:63:4a:e4:ee:d6 brd ff:ff:ff:ff:ff:ff, spoof checking on, link-state auto, trust off
    vf 1     link/ether ea:ad:a3:18:a3:17 brd ff:ff:ff:ff:ff:ff, spoof checking on, link-state auto, trust off
    vf 2     link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff, spoof checking on, link-state auto, trust off
    vf 3     link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff, spoof checking on, link-state auto, trust off
    
$ ip link show enp6s0f1
4: enp6s0f1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether a0:36:9f:a4:66:4b brd ff:ff:ff:ff:ff:ff
    vf 0     link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff, spoof checking on, link-state auto, trust off
    vf 1     link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff, spoof checking on, link-state auto, trust off
    vf 2     link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff, spoof checking on, link-state auto, trust off
    vf 3     link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff, spoof checking on, link-state auto, trust off
```

# 5. Ping between Virtual Machines
```
vagrant@sriov-0:~$ sudo ip a add 192.168.200.113/24 dev eth2
vagrant@sriov-0:~$ sudo ip link set eth2 up
```
```
vagrant@sriov-1:~$ sudo ip a add 192.168.200.114/24 dev eth2
vagrant@sriov-1:~$ sudo ip link set eth2 up
vagrant@sriov-1:~$ ping 192.168.200.114
PING 192.168.200.114 (192.168.200.114) 56(84) bytes of data.
64 bytes from 192.168.200.114: icmp_seq=1 ttl=64 time=0.039 ms
64 bytes from 192.168.200.114: icmp_seq=2 ttl=64 time=0.059 ms
^C
--- 192.168.200.114 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1010ms
rtt min/avg/max/mdev = 0.039/0.049/0.059/0.010 ms
```
