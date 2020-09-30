# Using the Ooma Wi-Fi &amp; Bluetooth adapter in Arch Linux

- git clone [https://aur.archlinux.org/8192cu-dkms.git](https://aur.archlinux.org/8192cu-dkms.git)
- cd 8192cu-dkms
- makepkg -s
- delete the created package
- (optional but recommended*): edit 8192cu-dkms/src/8192cu/os_dep/linux/usb_intf.c, add vendor and dev:
	{USB_DEVICE(0x226A, 0x817B)}, /* ooma wifi/bt dongle */ \
- makepkg -s 
- install the package
- insert the module: `# insmod /lib/modules/$(uname -r)/kernel/drivers/net/wireless/8192cu.ko`
- use `-D wext` with `wpa_supplicant`. iwd doesn't appear to work at all with this device.

 * if you don't do this, you'll have to do the following before loading the module:
  `# echo 226a 817b > /sys/bus/usb/drivers/rtl8192cu/new_id`

Use kernel-supplied (i.e. not 8192cu-dkms) modules if you need monitor mode
- `# rmmod 8192cu`
- `# modprobe rtl8192cu`
- `# ip link set $DEV down`
- `# iw $DEV set monitor control`
- `# ip link set $DEV up`
