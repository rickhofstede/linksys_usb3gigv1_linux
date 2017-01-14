# Linux driver for Linksys USB3GIGV1 (based on Realtek RTL8153)

This repository features a Linux driver for Linksys USB3GIGV1 USB 3.0 Ethernet adapters, which are based on Realtek's RTL8153.

## Introduction

One of the few available USB 3.0-based Ethernet adapters, the Linksys USB3GIGV1, is only supported officially on Windows and Mac devices. Although the adapter is also recognized as an Ethernet adapter on Linux (3.x and 4.x kernels), its support is far from mature. Some of the observed problems are the following:

* `ethtool` hardly reports any statistics:

```
# ethtool eth1
Settings for eth1:
         Current message level: 0x00000007 (7)
                                drv probe link
         Link detected: yes
```
* Capturing packets on the respective interface in promiscuous mode using `tcpdump` does not show any packets.
* `netstat -i` reports most Ethernet frames as errors.

These symptoms clearly indicate a driver problem/incompatibility. The default driver (i.e., kernel module) loaded on Ubuntu systems for this adapter is `cdc_ether`:
```
Apr 11 19:43:24 <host> kernel: [33110.208809] usb 2-3: new SuperSpeed USB device number 3 using xhci_hcd
Apr 11 19:43:24 <host> kernel: [33110.225299] usb 2-3: New USB device found, idVendor=13b1, idProduct=0041
Apr 11 19:43:24 <host> kernel: [33110.225303] usb 2-3: New USB device strings: Mfr=1, Product=2, SerialNumber=6
Apr 11 19:43:24 <host> kernel: [33110.225305] usb 2-3: Product: Linksys USB3GIGV1
Apr 11 19:43:24 <host> kernel: [33110.225306] usb 2-3: Manufacturer: Linksys
Apr 11 19:43:24 <host> kernel: [33110.225307] usb 2-3: SerialNumber: 000001000000
Apr 11 19:43:24 <host> kernel: [33110.226513] cdc_ether 2-3:2.0 eth1: register 'cdc_ether' at usb-0000:00:14.0-3, CDC Ethernet Device, <mac>
```

The Linksys USB3GIGV1 is based on the Realtek RTL8153, for which [Realtek provides Linux drivers](http://www.realtek.com/downloads/downloadsView.aspx?Langid=1&PNid=56&PFid=56&Level=5&Conn=4&DownTypeID=3&GetDown=false#RTL8153). For the Linux kernel to be able to connect drivers to hardware devices, the drivers need to carry the USB vendor ID and product ID of the hardware. However, since the Linksys adapter announces itself to the OS as a Linksys device (and not as a Realtek device), the Realtek driver will never be attached to the Linksys hardware.

The code in this repository solves the problem of a Linux system not connecting the Realtek driver to the Linksys adapter. This is done by including the Linksys USB vendor ID and product ID in the code, which is done [here](https://github.com/rickhofstede/linksys_usb3gigv1_linux/commit/53788c4b0dde3d7259fe81a7ae0abd8fd0708fcb).

## Installation instructions

To build the Linux driver, clone this repository first:
```
$ git clone https://github.com/rickhofstede/linksys_usb3gigv1_linux.git
```

Make sure your system features a build environment. This is typically provided by the `build-essential` meta-package. Then, execute the following commands:
```
$ cd linksys_usb3gigv1_linux
$ make
$ make install
```

You may need superuser permissions for executing the `make install` command. The following steps are used to ensure that the new driver is loaded, rather than the system-default/old one. Note that these commands are for Ubuntu and (minor) adaptations may be needed for other distributions.

Unload the existing module (e.g., `cdc_ether`):
```
$ modprobe -r cdc_ether
```

Load the new module (`r8152`):
```
$ modprobe r8152
```

Make sure the driver gets loaded on system boot by rebuilding initramfs:
```
$ update-initramfs -u
```

You may now either (re)connect your USB Ethernet adapter.

## Compatibility

This driver has been tested on Ubuntu 14.04 LTS with the 4.2.0-35 kernel.
