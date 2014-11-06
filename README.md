Huawei E5377 OpenWRT
====================

Huawei E5377 OpenWRT usb_modeswitch configuration and cdc_ether

Manual approach
--------------------

Connect your device to the OpenWRT box with USB cable. After that you should see in demsg:
```
usb 1-1: new high-speed USB device number xx using ehci-platform
```

At /sys/kernel/debug/usb/devices before mode-switch descriptors are:
```
Vendor=12d1 ProdID=1f02 Rev= 1.02
```

Manual from command line:
```
opkg update
opkg install kmod-usb-net
opkg install kmod-usb-net-cdc-ether
usb_modeswitch -v 12d1 -p 1f02 -M 55534243123456780000000000000a11062000000000000100000000000000
```

Then you should see in dmesg:
```
usb 1-1: usbfs: process xxxx (usb_modeswitch) did not claim interface 0 before use
usb 1-1: USB disconnect, device number xx
usb 1-1: new high-speed USB device number xx using ehci-platform
cdc_ether 1-1:1.0: eth1: register 'cdc_ether' at usb-ehci-platform-1, CDC Ethernet Device, xx:xx:xx:xx:xx:xx
```

At /sys/kernel/debug/usb/devices after mode-switch descriptors are:
```
Vendor=12d1 ProdID=14dc Rev= 1.02
```

Then use your eth1 network adapter as a WAN device (mode DHCP) just as it is any other network adapter.

Automating all this
--------------------

You MUST do steps above before continuing to this section!

Create a file:
```
/etc/usb_modeswitch.d/12d1:1f02
```
with content:
```
# Huawei E5377 (rev.1.02)

TargetVendor=  0x12d1
TargetProduct= 0x14dc

MessageContent="55534243123456780000000000000a11062000000000000100000000000000"
NoDriverLoading=1
```

Then reboot your OpenWRT device and check that everything starts as expected.
