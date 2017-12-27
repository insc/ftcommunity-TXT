This is a experimental fork of [ftCommunity/ftcommunity-TXT](https://github.com/ftCommunity/ftcommunity-TXT). If you want to install a stable txt firmware please use their project.


## (German) FTCommunity Forum 

Most discussions around the community firmware take place in the [FTCommunity forum)](http://forum.ftcommunity.de/viewforum.php?f=8). This is a german forum but english contributions are welcome.

# Usage

Two simple steps are needed to run the community firmware on your TXT:

 1. A correctly prepared micro SD card
 2. Configure the TXT to boot from SD card

Both steps are simple and risk-free as removing the SD card will always give you the original firmware back. 

# Step 1: Preparing the SD card

You'll need a micro SD card. A size of 2 or 4 GB is recommended.

## Use pre-built images

Pre-built images for a hassle free quick start have been [released](https://github.com/ftCommunity/ftcommunity-TXT/releases).

For beginners it's recommended to use the pre-built releases.

For the latest bleading-edge versions you might want to build the firmware yourself as explained below.

## Build the firmware with docker

See [docker-build-txt-firmare](https://github.com/insc/docker-build-txt-firmware)

## Prepare SD Card
You need an empty micro SD card for the ftcommunity firmware.

The ftcommunity firmware supports two different SD card layouts:
* simple layout: Everything is stored on a single FAT partition, the linux root file system is read-only and resides in an image file on the FAT partition. This is the recommended setup for most users.
* advanced layout: The linux root file system is read-write and stored on a separate partition. This is the recommended layout for developers.

In both layouts, user installed apps and persistent settings are stored on the FAT partition.

### Simple Layout
Make sure that the first partition on the SD card is formatted as FAT (most fresh SD cards should already have this layout). Then copy the files `output/images/uImage`, `output/images/am335x-kno_txt.dtb` and `output/images/rootfs.img` to the SD card.

### Advanced Layout
Create two partitions on the SD card. Both partitions should have a size of at least 100 MB, the recommended setup is to reserve ca. 200-500 MB for the second partition and allocate most space to the first partition. Format the first partition as `FAT` and the second partition as `ext4`.

The following commands will do this on a linux system where the SD card slot is named `/dev/mmcblk0`. *Make sure this is really the empty SD card, the following commands will destroy all data on `/dev/mmcblk0`*:

```
parted /dev/mmcblk0 mklabel msdos
parted -a optimal /dev/mmcblk0 -- mkpart primary fat32 1MB -300MB
parted -a optimal /dev/mmcblk0 -- mkpart primary ext2 -300MB 100%
parted /dev/mmcblk0 set 1 boot on
mkfs.vfat -n BOOT /dev/mmcblk0p1
mkfs.ext4 -L ROOT /dev/mmcblk0p2
```

Now, copy the files `output/images/uImage` and `output/images/am335x-kno_txt.dtb` to the first partition of the SD card:
```
mount /dev/mmcblk0p1 /mnt
cp output/images/uImage output/images/am335x-kno_txt.dtb /mnt/
umount /mnt
```

Finally, unpack `output/images/rootfs.tar` to the second partition on the SD card:
```
mount /dev/mmcblk0p2 /mnt
tar xvf output/images/rootfs.tar -C /mnt/
umount /mnt
```

Note: On most linux systems, you will need to run all of these commands as root.

# Step 2: Configure the TXT to boot from SD card

Make sure you are running at least version [4.2.4 of the official firmware](http://www.fischertechnik.de/home/downloads/Computing.aspx). You then only need to follow the [instructions provided by Fischertechnik](http://www.fischertechnik.de/ResourceImage.aspx?raid=10278).

More details and a brief set of english instructions can be found in our [Wiki](https://github.com/ftCommunity/ftcommunity-TXT/wiki/Preparing-the-TXT-controller).

# Run the firmware

Switch off the TXT, insert the SD card with the ftcommunity firmware in the SD card slot and restart the TXT. 

The ftcommunity firmware enables ssh login via USB by default, network settings for USB are the same as with the original firmware: The TXT has address 192.168.7.2, the host computer has address 192.168.7.1. On most systems, the host computer address should be set up automatically when you connect the USB cable.

You can log in to the TXT either via [serial console](https://github.com/ftCommunity/ftcommunity-TXT/wiki/Serial-Console) or using ssh. 

In both cases, the user name is "txt". By default, the *txt* user can log in without a password. 

After setting a password, the *txt* user may use `sudo` to execute commands as root.
