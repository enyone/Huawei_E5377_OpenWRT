Huawei E5377 OpenWRT
====================

Huawei E5377 OpenWRT usb_modeswitch configuration and cdc_ether

Manual approach
--------------------

Connect your device to the OpenWRT box with USB cable. After that you should see in dmesg:
```
usb 1-1: new high-speed USB device number xx using ehci-platform
```

At /sys/kernel/debug/usb/devices before mode-switch descriptors are:
```
Vendor=12d1 ProdID=1f02 Rev= 1.02
```

At command line (SSH to OpenWRT):
```
opkg update
opkg install usb-modeswitch usb-modeswitch-data kmod-usb-net kmod-usb-net-cdc-ether
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

Then use your eth1 interface as a WAN device (mode DHCP) just as it is any other network adapter.

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

It seems that OpenWRT 15.05 uses a file named "/etc/usb-mode.json". In that case, add the following section to it:
```
                "12d1:1f02": {
                        "*": {
                                "t_vendor": 4817,
                                "t_product": [ 5339 ],
                                "mode": "HuaweiNew",
                                "no_driver": true,
                                "msg": [  ]
                        }
                },
```
Also, usb-modeswitch-data didn't see to be available on 15.05, but didn't seem to be necessary.
