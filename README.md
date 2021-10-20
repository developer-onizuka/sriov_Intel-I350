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
