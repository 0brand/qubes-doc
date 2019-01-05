---
layout: doc
title: Block or Storage Devices in Qubes R4.0
permalink: /doc/block-devices/
redirect_from:
- /doc/block-devices-in-qubes-R4.0/
---

Block or Storage Devices in Qubes R4.0
======================================
*This page is part of [device handling in qubes]*
(In case you were looking for the [R3.2 documentation](/doc/usb/).)

If you don't know what a "block device" is, just think of it as a fancy way to say "something that stores data".

#Using The GUI to Attach a Drive
(**Note:** In the present context, the term "USB drive" denotes any [USB mass storage device][mass-storage].
In addition to smaller flash memory sticks, this includes things like USB external hard drives.)

Qubes OS supports the ability to attach a USB drive (or just its partitions) to any qube easily, no matter which qube handles the USB controller.

Attaching USB drives is integrated into the Devices Widget: ![device manager icon]  
Simply insert your USB drive and click on the widget.
You will see multiple entries for your USB drive; typically, `sys-usb:sda`, `sys-usb:sda1`, and `sys-usb:2-1` for example.
Entries starting with a number (e.g. here `2-1`) are the [whole usb-device][USB]. Entries without a number (e.g. here `sda`) are the whole block-device. Other entries are partitions of that block-device (e.r. here `sda1`).

The simplest option is to attach the entire block drive.
In our example, this is `sys-usb:sda`, so hover over it.
This will pop up a submenu showing running VMs to which the USB drive can be connected.
Click on one and your USB drive will be attached!

**Note:** attaching individual partitions (e.g. `sys-usb:sda1`) can be slightly more secure because it doesn't force the target AppVM to parse the partition table.
However, it often means the AppVM won't detect the new partition and you will need to manually mount it inside the AppVM.
See below for more detailed steps.

#Block Devices in VMs
If not specified otherwise, block devices will show up as `/dev/xvdi*` in a linux VM, where `*` may be the partition-number. If a block device isn't automatically mounted after attaching, open a terminal in the VM and execute:

    cd ~
    mkdir mnt
    sudo mount /dev/xvdi2 mnt

where `xvdi2` needs to be replaced with the partition you want to mount.
This will make your drive content accessible under `~/mnt`.

Beware that when you attach a whole block device, partitions can be identified by their trailing integer (i.e. `/dev/xvdi2` for the second partition, `/dev/xvdi` for the whole device), whereas if you attach a single parition, the partition has *no trailing integer*. <!--TODO: really? Didn't have a drive to test this quickly.-->

If several different block-devices are attached to a single VM, the last letter of the device node name is advanced through the alphabet, so after `xvdi` the next device will be named `xvdj`, the next `xvdk`, and so on.

To specify this device node name, you can pass `--option frontend-dev=[custom-node-name]` to `qvm-block attach`.

#Command Line Tool Guide
The command-line tool you may use to mount whole USB drives or their partitions is `qvm-block`, a shortcut for `qvm-device block`.

`qvm-block` only sees device-nodes and may not use names you expect! So make sure to have the drive available in the sourceVM before listing available block devices (step 1.) to find out it's its ID.

In case of a USB-drive, make sure it's attached to your computer. If you don't see anything that looks like your drive, run `sudo udevadm trigger --action=change` in your USB-qube (typically `sys-usb`)

 1. In a dom0 console (running as a normal user), list all available block devices:
    
        qvm-block
    
    This will list all available block devices in your system across all VMs, no matter whether hosted by a USB controller or `losetup`.
    The name of the qube hosting the block device is displayed before the colon in the device ID.
    The string after the colon is the ID of the device used within the qube, like so:

        sourceVM:sdb     Cruzer () 4GiB
        sourceVM:sdb1    Disk () 2GiB

 2. Assuming your block device is attached to `sys-usb` and its device node is `sdb`, we attach the device to a qube with the name `work` like so:
    
        qvm-block attach work sys-usb:sdb
    
    This will attach the device to the qube as `/dev/xvdi` if that name is not already taken by another attached device, or `/dev/xvdj`, etc.
    
    You may also mount one partition at a time by using the same command with the partition number, e.g. `sdb1`.

 3. The block device is now attached to the qube.
    If using a default qube, you may open the Nautilus file manager in the qube, and your drive should be visible in the **Devices** panel on the left.
    If you've attached a single partition (e.g. `sdb2` instead of `sdb` in our example), you may need to manually mount before it becomes visible:
    
        cd ~
        mkdir mnt
        sudo mount /dev/xvdi mnt
    

 4. When you finish using the block device, click the eject button or right-click and select **Unmount**.

    If you've manually mounted a single partition in the above step, use:

        sudo umount mnt

 5. In a dom0 console, detach the device

        qvm-block detach work sys-usb:sdb

 6.  You may now remove the device or attach it to another qube.
<!--TODO: what happens if USB-device removed before detaching? How about attaching a blockdevice to two qubes?-->

#Attaching a File
To attach a file as block device to another qube, first turn it into a loopback device inside the sourceVM.

 1. In the linux sourceVM run

        sudo losetup /dev/loop0 /path/to/file

    (increase the trailing integer if `loop0` is already in use or use other name. This is just a generic device-id.)

 2. If you want to use the GUI, you're done. Click the Device Manager ![device manager icon] and select the `loop0`-device to attach it to another qube.

    If you rather use the command line, continue:

    In dom0, run `qvm-block` to display known block devices. The newly created loop device should show up:

        ~]$ qvm-block
        BACKEND:DEVID  DESCRIPTION  USED BY
        sourceVM:loop0 /path/to/file

 3. Attach the `loop0`-device using qvm-block as usual:

        qvm-block a targetVM sourceVM:loop0

 4. After detaching, destroy the loop-device inside the sourceVM as follows:

        sudo losetup -d /dev/loop0

#Additional Attach Options
Attaching a block device through the command line offers additional customisation options, specifiable via the `--option`/`-o` option. (Yes, confusing wording, there's an [issue for that](https://github.com/QubesOS/qubes-issues/issues/4530).)

##frontend-dev
This option allows you to specify the name of the device node made available in the targetVM. This defaults to `xvdi` or, if already occupied, the first available device node name in alphabetical order. (The next one tried will be `xvdj`, then `xvdk`, and so on ...)

usage example:

    qvm-block a work sys-usb:sda1 -o frontend-dev=xvdz

This command will attach the partition `sda1` to `work` as `/dev/xvdz`.

##read-only
Attach device in read-only mode. Protects the block device in case you don't trust the targetVM.

If the device is a read-only device, this option is forced true.

usage example:

    qvm-block a work sys-usb:sda1 -o read-only=true

There exists a shortcut to set read-only `true`, `--ro`:

    qvm-block a work sys-usb:sda1 --ro

The two commands are equivalent.

##devtype
Usually, a block device is attached as disk. In case you need to attach a block device as cdrom, this option allows that.

usage example:

    qvm-block a work sys-usb:sda1 -o devtype=cdrom

This option accepts `cdrom` and `disk`, default is `disk`.



[device handling in qubes]: /doc/device-handling/
[mass-storage]: https://en.wikipedia.org/wiki/USB_mass_storage_device_class
[device manager icon]:https://raw.githubusercontent.com/hrdwrrsk/adwaita-xfce-icon-theme/master/Adwaita-Xfce/22x22/devices/media-removable.png <!--TODO: find actual icon used in qubes!-->
[USB]:/dock/usb-devices-in-qubes-R4.0/