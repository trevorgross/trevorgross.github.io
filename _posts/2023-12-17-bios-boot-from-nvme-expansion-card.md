---
layout: post
title: 'Boot from NVMe PCIe expansion card with an old BIOS'
---
[Skip to how to do this](#steps)

Recently I installed an NVMe drive into a PCIe slot using a cheap NVMe-PCIe adapter. The motherboard it's intalled on is old and has a legacy non-UEFI BIOS. 

The adapter works great. The BIOS shows "expansion cards" or something similar as a boot option. All I had to do was select that boot option and I could use the computer with just the NVMe and not other drives.

Except it didn't work. It wouldn't boot from the NVMe expansion card. Maybe that option exists because other types of expansion cards work. Or maybe it just doesn't work at all. It wouldn't surprise me. 

Anyway, no problem, I'll just throw a USB thumb drive in, install `/boot` to the thumb drive, and be on my way. As you already know this also didn't work. 

The basic concept of "`/boot` on the thumb drive" was sound, but it requires additional partitions and some arcane flags. The steps, when installing Arch Linux, are as follows. Note that you can use an SATA drive instead of a USB drive if you adjust the steps accordingly.

<div id="steps"></div>

## Step-by-step Instructions

1. Install NVMe PCIe expansion card
2. Insert USB drive.
    1. Consider trying multiple USB ports until the system sees it as `/dev/sda`
    2. The drive can be small; I'm using a 256Mb drive
3. Boot the Arch installer from another USB drive
4. Format the NVMe using GPT partition table
    1. GPT is required for the USB drive, may or may not be required here, but just use it
5. Format the USB drive using GPT partition table
    1. This one *MUST* be GPT
    2. Create two partitions:
        1. `/boot`, of whatever size you want. I'm using only 228Mb.
        2. Unformatted partition, can be very small. I'm using 10Mb. Partition type probably doesn't matter. I'm using EF02, which is a UEFI partition type, and this computer doesn't even have UEFI.
    3. Write the partition table of the USB drive and exit
    4. Use parted to set the following flags:
    5. Legacy boot on the /boot partition:

        `(parted) set 1 legacy_boot on`
    6. bios_grub, whatever that is, on the second partition:

        `(parted) set 2 bios_grub on`
6. Mount the `/boot` partition of the USB drive, `/dev/sda1`, on `/mnt/boot`. Install Arch as usual.
7. Install grub to the USB drive with `grub-install --target=i386-pc /dev/sda`
    1. Install to the raw device (sda), Do **not** use a partition (sda1).
8. Configure BIOS to boot from USB drive

## References
This post is what finally got me there: <https://ubuntuforums.org/showthread.php?t=2415658&p=13849399#post13849399>

Relevant `parted` manual: <https://www.gnu.org/software/parted/manual/html_node/set.html>
