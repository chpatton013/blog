# Installing Arch Linux on a Pinebook Pro (with full-disk encryption)

*2020/06/23*

Below is the story of how I spent my whole weekend installing Arch Linux on a
Pinebook Pro with full-disk encryption. If you are not interested in the story,
feel free to skip ahead to [the instructions](#the-instructions).

## The Story

### This sounds like fun

A couple months ago I received the [Pinebook Pro][Pine64-PinebookPro] that I
forgot I had ordered. When it arrived I turned it on to make sure it worked,
powered it back off, and immediately forgot about it - dooming it to a fate of
being slowly buried on my desk.

And then I cleaned my desk! I was very proud of myself, and excited to
uncover this forgotten laptop. I needed a distraction this weekend, so I decided
that I would install [Arch Linux][ArchLinux] on it. That shouldn't be too hard;
I've done this sort of thing before. I just need to find out of the Pinebook Pro
(or PBP) uses [BIOS][Wikipedia-BIOS] or [UEFI][ArchWiki-UEFI], and then...

### Complication #1: The boot firmware

Well it turns out that the PBP doesn't use either of the major boot firmware
solutions that I was familiar with. And in retrospect, I shouldn't have expected
it to. The PBP is essentially a [Single Board
Computer][Wikipedia-SingleBoardComputer] (or SBC) with some extra peripherals
and a case.

Instead, I should be looking for tutorials or posts from people who have
installed Arch Linux on older SBCs made by the Pine64 folks. I'm sure they start
by copying an Arch Linux Installer ISO to a flash drive or SD card or something.
I'll just do that to get started. That will take a few minutes to write, so I'll
do some research while I wait. Hmm, seems like everyone gets their ISOs from the
[Arch Linux ARM downloads page][ArchLinuxArm-Downloads]...

### Complication #2: The processor architecture

I guess I should have seen that one coming too. The whole reason I wanted this
machine in the first place was because it was all open-source hardware. So of
course it's not going to have an x86-64 processor like I'm used to. At this
point I started to realize that I was operating very far out of my wheelhouse.

So let's rewind here and scrub all that confidence I had going into this
project. Instead of charging in without doing the requisite research, I'll start
over and treat this like it's my first time installing an operating system. Time
to turn to Google for the basics: "install arch linux on pinebook pro".

### Complication #3: Arch Linux is not supported

But [Manjaro][] is. Apparently it's so well-supported that they started shipping
the machines with a fully-functioning Manjaro installation. I don't know much
about Manjaro, but I do know that it's an Arch Linux derivative. So that means
that there's hope!

I found several Google results that reinforced the idea. People *were* talking
about this, but I couldn't find a success story - let alone a tutorial. That is,
until I found [Rudis Muiznieks' blog][RudisM]. Rudis, If you ever read this,
thank you. I likely would have given up on this project if I hadn't found your
blog.

### Getting back on track

I read through Rudis' blog post and found that Rosetta stone I would need to see
this project through. He linked to a [Github Repo][Github-Archiso-Pbp] by Nadia
Holmquist Pedersen (or NHP, for brevity) that contains a customized archiso for
the PBP. A huge thank-you to Nadia as well. I certainly would have failed if I
hadn't found your repository.

This link was important for several reasons:
1. It gave me a way to boot into a live installer, where I could leverage my
   existing knowledge about installing a Linux system.
1. It showed me how to make media that was bootable by the PBP.
1. It referenced some custom packages from a [pacman user
   repository][NHP-PinebookPro] that NHP hosts, which tipped me off that there
   was going to be some custom firmware I would need to install.

### Learning about boot firmware

NHP's ArchISO repository gave me the pointers I needed to learn about the PBP's
boot sequence. I tracked down the [Rockchip wiki][RockchipWiki-BootOption] and
learned all about the boot process for their chips. This is my summary of the
process:

> The Rockchip BootRom will invoke the Secondary Program Loader (SPL, also
> called the "pre-loader" or "loader1") found at sector 0x40 on the boot device.
> The purpose of this program is to define (or ID) the Trusted Execution
> Environment (TEE) parameters before yielding to the next boot stage.
>
> The SPL then invokes the Tertiary Program Loader (TPL, also called the
> "boot-loader" or "loader2") found at sector 0x4000 on the boot device. The
> purpose of this program is to initialize the ramdisk with the trusted
> bootloader.
>
> The TPL then searches for a filesystem on a partition found at sector 0x8000
> on the boot device, also called the "boot partition". The filesystem on this
> partition must have a configuration file that UBoot will read to identify the
> kernel image (also a file on the same filesystem). The TPL invokes this kernel
> image, which later completes the boot sequence.

### Making it harder than it needed to be

Once I was up to speed on what it took to make a bootable ArchISO for the PBP, I
had my confidence back. No longer was this some mysterious device that required
a sacred rite to work - it just needed a bootloader and kernel image in the
right places. Surely that means that I can do anything on this machine that I
could do on my other Linux machines. So what about [full-disk
encryption][ArchWiki-FullDiskEncryption]? Better question: why not full-disk
encryption? Trick question: always full-disk encryption.

### Powering through

So I burned the ArchISO to an SD card, booted the machine, and started
installing Arch Linux:
* I formatted the eMMC with the partitions and volumes I wanted.
* I installed the packages I would need (including all the special ones from
  NHP's repository).
* I configured all the system-level settings that systemd manages.

And as I prepared to generate the initramfs, I stopped to make sure I was doing
this part right. This is where things could go wrong, since we're loading the
kernel and the firmware now. The [Arch
Wiki][ArchWiki-FullDiskEncryption-Configuring_mkinitcpio] spells out the hooks
you need in an encrypted system, but it doesn't say anything about the modules
you'll need for an unsupported machine.

The Pinebook Pro is not perfectly-well-supported by mainline-Linux yet, so we
will need to do a little bit of hand-holding in the process of selecting
modules. But fortunately, this is where I could turn back to Rudis' blog. He had
already done the hard work of tracking down [the modules][RudisM-LuksEncryption]
that would be sufficient to support a graphical decryption prompt from [an
earlier Pine64 forum post][Pine64-Forum]. I did several hours of my own research
to find out if these modules were all necessary, but eventually decided that I
didn't know what I was looking for and the effort of incrementally disabling
modules one at a time wasn't that interesting right now.

### Finishing up

This leaves us with one final step: writing the bootloader to the device. A
couple of `dd` commands and a syslinux config-file later, and we're all done!
Except it wasn't quite that easy. I spent quite a while trying to find out why I
was presented with errors instead of a decryption prompt, only to eventually
learn that the order of command-line parameters to the kernel were very
important.

I referred to the [Arch Wiki][ArchWiki-KernelParameters] page on relevant kernel
parameters to make sure there weren't any that I was missing. And I briefly
entertained the notion of reviewing [all the available
parameters][Kernel-Parameters], but the number of them scared me off pretty
quickly.

## The Instructions

### Step 0: Getting Ready to Install

We are going to stick as closely as we can to the normal steps laid out in the
[Arch Linux Installation Guide][ArchWiki-InstallationGuide]. You should
familiarize yourself with its contents before we begin. I will call out any
differences to the guide as we proceed.

We're using [NHP's ArchISO][Github-Archiso-Pbp] as our live install media.
Download the latest release and `dd` it to an SD card:

```
dd if=archlinux-*-pbp.img of=/dev/sdcard
sync
```

Once that finishes, boot off the SD card.

Installation will require an internet connection. `dhcpd` should already be
running on the ArchISO, so if you have connected a wired network interface you
should be fine. But if you want to use wireless you will need to connect
explicitly and acquire a DHCP lease manually:

* Open networks: `iw dev wlan0 connect "SSID"`
* WEP networks: `iw dev wlan0 connect "SSID" key "0:KEY"`
* WPA networks: `wpa_supplicant -B -iwlan0 -c <(wpa_passphrase "SSID" "KEY")`
* DHCP: `dhclient`

And before we begin doing anything to the destination machine, update the system
clock:

```
timedatectl set-ntp true
```

### Step 1: Prepare Installation Media

We will prepare the install media with an encrypted root partition, and
unencrypted boot partition, and enough space at the beginning of the device to
host the bootloader images. The resulting drive should look like this:

```
/dev/mmcblk2
 |\- 0x00000000: GPT
 |\- 0x00000040: SPL
 |\- 0x00004000: TPL
 |\- 0x00008000: Boot Partition: 128MB
 |    \- FS: ext4: /boot
  \- 0x00048800 Root Partition: Remaining Space
      \- LUKS volume: cryptroot
          \- FS: ext4: /
```

I ran the following commands to make my system match the diagram:
```
parted --script -- /dev/mmcblk2 mklabel gpt
parted --script -- /dev/mmcblk2 unit s mkpart primary 32768 294912 name 1 boot
parted --script -- /dev/mmcblk2 unit s mkpart primary 296960 100% name 2 root
cryptsetup --batch-mode luksFormat --type luks2 /dev/mmcblk2p2
cryptsetup --batch-mode open /dev/mmcblk2p2 cryptroot
mkfs --type=ext4 -F /dev/mmcblk2p1
mkfs --type=ext4 -F /dev/mapper/cryptroot
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/mmcblk2p1 /mnt/boot
mkdir /mnt/etc
genfstab -U /mnt >/mnt/etc/fstab
```


### Step 2: Install Packages

We will install the packages we need to have an operational system.
Traditionally this set is just `base` (which is what makes this an ArchLinux
system), `linux` (the kernel), `linux-firmware` (the firmware for your version
of the kernel), and whichever bootloader you want to use. However, we are going
to need to make a few changes to that for the Pinebook Pro, specifically by
including some alternate packages from [NHP's user repository][NHP-PinebookPro].

* First, we need to make sure we have the `cryptsetup` package installed so we
  can build an initramfs capable of decrypting our root partition.
* Next, we need the kernel (and its headers) tailored to the Pinebook Pro. We
  will use the `linux-pbp` package.
* Likewise, there are a few Pinebook Pro firmware packages that
  `linux-firmware` is missing.
* The Pinebook Pro makes the choice of bootloader for us: we have to use
  UBoot. However just like several other packages, there is a specialized
  version of UBoot for the Pinebook Pro: `uboot-pbp`.

You can install all of those with the following command:
```
pacstrap /mnt base cryptsetup linux-pbp linux-pbp-headers linux-firmware ap6256-firmware linux-atm pbp-keyboard-hwdb pinebookpro-audio uboot-pbp
```

### Step 3: Configure System

This step is essentially identical to the installation guide. We set up our
timezone, localization, and network settings with the following commands:

```
systemd-firstboot --setup-machine-id --timezone=America/Los_Angeles --locale=en_US.UTF-8 --keymap=us --hostname=arch-pbp --root-password=hunter2 --root=/mnt

ln --symbolic --force /usr/share/zoneinfo/America/Los_Angeles /mnt/etc/localtime
arch-chroot /mnt hwclock --systohc

echo en_US.UTF-8 $charset >/mnt/etc/locale.gen
echo LANG=en_US.UTF-8 >/mnt/etc/locale.conf
echo KEYMAP=us >/mnt/etc/vconsole.conf
arch-chroot /mnt locale-gen

echo arch-pbp >/mnt/etc/hostname
cat >/mnt/etc/hosts <<EOF
127.0.0.1 localhost
::1       localhost
127.0.1.1 arch-pbp.localdomain arch-pbp
EOF
```

Note that you can find your timezone (and other geo-ip information) with the
following [API endpoint](http://ip-api.com/json).

### Step 4: Create Initramfs

We need to customize the configuration for `mkinitcpio`, then invoke it to
regenerate our initramfs. We are going to need to specify some custom hooks and
modules to support full-disk encryption and the PBP firmware. Order for both of
these is crucial; do not change it. I've annotated the list of modules with a
description of what they are for.

Run the following command to populate the config file and generate the ramfs:

```
cat >/mnt/etc/mkinitcpio.conf <<EOF
MODULES=(
  panfrost          # Panfrost (DRM support for ARM Mali Midgard/Bifrost GPUs)
  rockchipdrm       # DRM Support for Rockchip
  hantro_vpu        # Hantro VPU driver
  analogix_dp       # Analogix Display Port driver
  rockchip_rga      # 2D Graphics Hardware Acceleration
  panel_simple      # DRM panel driver for dumb panels
  arc_uart          # ARC UART driver support
  cw2015_battery    # CW2015 Battery driver
  i2c-hid           # HID over I2C transport layer
  iscsi_boot_sysfs  # iSCSI Boot Sysfs Interface
  jsm               # Digi International NEO and Classic PCI Support
  pwm_bl            # Simple PWM based backlight control
  uhid              # User-space I/O driver support for HID subsystem
)
BINARIES=()
FILES=()
HOOKS=(
  base
  udev
  keyboard
  autodetect
  keymap
  modconf
  block
  encrypt
  filesystems
  fsck
)
COMPRESSION="xz"
EOF
arch-chroot /mnt mkinitcpio −−allpresets
```

### Step 5: Install Bootloader

We use `dd` to write two different images to specific disk sectors on the boot
device, and then populate the Syslinux `extlinux.conf` file with relative paths
to the kernel image, the flattened device tree, and the Linux command-line. That
command-line is yet-another list of strings you have to get just right or your
system won't boot.

Run the following commands to write the bootloader images and config file:

```
dd if=/mnt/boot/idbloader.img of=/dev/mmcblk2 seek=64 conv=notrunc
dd if=/mnt/boot/u-boot.itb of=/dev/mmcblk2 seek=16384 conv=notrunc

cat >/mnt/boot/extlinux/extlinux.conf <<EOF
LABEL Arch Linux ARM
KERNEL /Image
FDT /dtbs/rockchip/rk3399-pinebook-pro.dtb
APPEND initrd=/initramfs-linux.img console=tty1 cryptdevice=/dev/mmcblk2p2:cryptroot root=/dev/mapper/cryptroot rw rootwait video=eDP-1:1920x1080@60
EOF
```

### Step 6: Cleanup. Oh yeah, and Profit

You should take this opportunity to install any other software you know you will
want on your machine. Making sure you have everything you need for your network
configuration is particularly important. You can invoke `pacstrap` again to do
this most-easily:

```
pacstrap /mnt list of packages you want
```

Or you can chroot into system using `arch-chroot` and do whatever you want in
there:

```
arch-chroot /mnt
```

When you're done, run the following commands to safely tear down the
installation environment:

```
sync
umount /mnt/boot
umount /mnt
cryptsetup --batch-mode close cryptroot
```

Then you can shutdown the machine, remove the SD card, and power it back on. If
all went well, you should be looking at the standard Linux boot-up sequence
followed by an all-too-familiar decryption prompt.

Enjoy your open-source, fully-encrypted hardware!

[ArchLinuxArm-Downloads]: https://archlinuxarm.org/about/downloads
[ArchLinux]: https://www.archlinux.org/
[ArchWiki-FullDiskEncryption-Configuring_mkinitcpio]: https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Configuring_mkinitcpio
[ArchWiki-FullDiskEncryption]: https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system
[ArchWiki-InstallationGuide]: https://wiki.archlinux.org/index.php/Installation_guide
[ArchWiki-KernelParameters]: https://wiki.archlinux.org/index.php/Kernel_parameters
[ArchWiki-UEFI]: https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface
[Github-Archiso-Pbp]: https://github.com/nadiaholmquist/archiso-pbp/
[Kernel-Parameters]: https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html
[Manjaro]: https://manjaro.org/
[NHP-PinebookPro]: https://nhp.sh/pinebookpro/
[Pine64-Forum]: https://forum.pine64.org/showthread.php?tid=9052
[Pine64-PinebookPro]: https://www.pine64.org/pinebook-pro/
[RockchipWiki-BootOption]: http://opensource.rock-chips.com/wiki_Boot_option
[RudisM-LuksEncryption]: https://rudism.com/installing-arch-linux-on-the-pinebook-pro/#Addendum-2020-06-09-LUKS-Encryption
[RudisM]: https://rudism.com/installing-arch-linux-on-the-pinebook-pro/
[Wikipedia-BIOS]: https://en.wikipedia.org/wiki/BIOS
[Wikipedia-SingleBoardComputer]: https://en.wikipedia.org/wiki/Single-board_computer
