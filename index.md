# Using the Ooma Wi-Fi &amp; Bluetooth adapter in Arch Linux

I got an Ooma Telo and the Wi-Fi &amp; Bluetooth adapter. Performance was abysmal with the adapter so I plugged an ethernet cable into the Telo. This left me with the Wi-Fi/BT dongle.

Ooma says of the dongle: "Works only with Ooma Telo. Not for use with other devices." So I plugged it into my laptop running Arch Linux and sure enough, it didn't work. Turns out the dongle is actually an internal USB hub with the Wi-Fi and Bluetooth attached to the hub. As such, the OS can't find the Wi-Fi adapter to load the module. The solution is to somehow indicate what the device is, either by sysfs or editing a file before compiling the module. 

- git clone [https://aur.archlinux.org/8192cu-dkms.git](https://aur.archlinux.org/8192cu-dkms.git)
- cd 8192cu-dkms
- makepkg -s
- delete the created package
- optional (but recommended*): edit 8192cu-dkms/src/8192cu/os_dep/linux/usb_intf.c, add vendor and dev:  
`{USB_DEVICE(0x226A, 0x817B)}, /* ooma wifi/bt dongle */ \`
- makepkg -s 
- install the package
- insert the module:  
`# insmod /lib/modules/$(uname -r)/kernel/drivers/net/wireless/8192cu.ko`
- use `-D wext` with `wpa_supplicant`. iwd doesn't appear to work at all with this device.

\* if you don't do this, you'll have to do the following every time before loading the module:  
 `# echo 226a 817b > /sys/bus/usb/drivers/rtl8192cu/new_id`

Use kernel-supplied (i.e. not 8192cu-dkms) modules if you need monitor mode
- `# rmmod 8192cu`
- `# modprobe rtl8192cu`
- `# ip link set $DEV down`
- `# iw $DEV set monitor control`
- `# ip link set $DEV up`
