# Using the Ooma Wi-Fi &amp; Bluetooth adapter in Arch Linux

tl;dr: use the 8192cu-dkms package and set the correct USB Vendor ID. 

I bought an Ooma Telo and the Wi-Fi &amp; Bluetooth adapter. Performance was abysmal with the adapter so I plugged an ethernet cable into the Telo. This left me with the Wi-Fi/BT dongle.

Ooma says of the dongle: "Works only with Ooma Telo. Not for use with other devices." 

Arch Linux didn't recognize the adapter(s). The dongle appears as a USB 2.0 "Genesys Logic, Inc. Hub" with ID 05e3:0608. Two other devices show up in `lsusb`. These are the Wi-Fi and Bluetooth adapters attached to the dongle's internal hub.  
```
Bus 001 Device 049: ID 226a:0001  
Bus 001 Device 048: ID 226a:817b  
Bus 001 Device 047: ID 05e3:0608 Genesys Logic, Inc. Hub  
```

Device 49 is the Bluetooth adapter, per dmesg it's a CSR8510 A10. I had Bluetooth enabled already, and the adapter just showed up. So btusb and probably some other module (btrtl, etc) got that working without me doing anything. 

Wi-Fi didn't work. I don't remember how but somehow I learned that the adapter is a Realtek 8192cu. Probably by Googling for "realtek idProduct=817b," or something along those lines. 

Loading the `rtl8192cu` module doesn't result in a working adapter. First, you have to set the `new_id` in sysfs to identify the device on the hub because it uses a nonstandard vendor id. Once you do that, the adapter shows up in `iwconfig`, etc. but you can't see or connect to a network or do anything useful. 

Using the 8192cu module from AUR works.

- git clone [https://aur.archlinux.org/8192cu-dkms.git](https://aur.archlinux.org/8192cu-dkms.git)
- cd 8192cu-dkms
- makepkg -s
- delete the created package
- optional (but recommended*): edit src/8192cu/os_dep/linux/usb_intf.c, add vendor and dev somewhere:  
`{USB_DEVICE(0x226A, 0x817B)}, /* ooma wifi/bt dongle */ \`
- makepkg -s 
- install the package
- insert the module:  
`# insmod /lib/modules/$(uname -r)/kernel/drivers/net/wireless/8192cu.ko`  
(or just `# modprobe 8192cu`)  
- use `-D wext` with `wpa_supplicant`. iwd doesn't appear to work at all with this device.

\* if you don't do this, you'll have to do the following every time:  
 `# echo 226a 817b > /sys/bus/usb/drivers/rtl8192cu/new_id`

Use kernel-supplied module, `rtl8192cu` if you need monitor mode
- `# rmmod 8192cu`
- `# modprobe rtl8192cu`
- `# echo 226a 817b > /sys/bus/usb/drivers/rtl8192cu/new_id`  
(if you haven't already)
- `# ip link set $DEV down`
- `# iw $DEV set monitor control`
- `# ip link set $DEV up`
