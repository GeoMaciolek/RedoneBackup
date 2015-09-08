# Redone Backup

## Overview

This is a clone of the source code for [Redo Backup](http://www.redobackup.org) from its [sourceforge page](http://sourceforge.net/projects/redobackup/)

It is intended to become a fork & continuation.

## Building

### Redone Backup / Clone Notes

**Note!  The following is untested!**  It is copied verbatim from the [original SourceForge documentation](http://sourceforge.net/p/redobackup/wiki/Home/).  Beware possible formatting issues; if in doubt, check the original docs, and always use care.  The documentation below refers to the project as **Redo Backup**," which is (or may be) copyrighted / trademarked by the original authors.  No claim of ownership is intended here.

### Creating a Redo Backup Live CD from Scratch

These steps will create a Live CD from scratch using debootstrap in about 30 to 45 minutes, depending on processor and download speeds. Adapted from https://help.ubuntu.com/community/LiveCDCustomizationFromScratch with a few minor modifications.

### Notes

As of the last revision of this document:

* Ubuntu release 12.04 (Precise)i
* ADeskBar version 0.4.3 (Available from http://adeskbar.tuxfamily.org/)

The latest version of Ubuntu may work fine using the instructions below, but you would need to replace the word "precise" in each instance with the latest release name in order to do so successfully. Using more recent Ubuntu builds may improve hardware support, but will likely result in a larger ISO image.

### 1. Prepare a base system
* sudo su
* **Create and enter a working directory of your choice such as "~/work/" before continuing!**
* `apt-get install debootstrap; mkdir -p livecd/chroot; cd livecd; debootstrap --arch=i386 precise chroot http://archive.ubuntu.com/ubuntu/`

Depending on your internet connection, the last step may take 5-10 minutes to download and unpack the base system.

### 2. Enter the chroot

* `mount --bind /dev chroot/dev; cp /etc/hosts chroot/etc/hosts; cp /etc/resolv.conf chroot/etc/resolv.conf; echo 'deb http://archive.ubuntu.com/ubuntu precise main universe multiverse' > chroot/etc/apt/sources.list; chroot chroot/`

### 3. Prepare the chroot environment

* `mount none -t proc /proc; mount none -t sysfs /sys; mount none -t devpts /dev/pts; export HOME=/root; export LC_ALL=C; apt-get update; apt-get install --no-install-recommends dbus; dbus-uuidgen > /var/lib/dbus/machine-id; dpkg-divert --local --rename --add /sbin/initctl; ln -s /bin/true /sbin/initctl`

### 4. Install the basic packages

Make cursor keys work in vi editor, make the local time show, and install the base system (about 65MB):

* `perl -p -i -e 's/^set compatible$/set nocompatible/g' /etc/vim/vimrc.tiny`
* `rm -rf /etc/network/if-up.d/ntpdate`
* `apt-get install --no-install-recommends discover laptop-detect os-prober linux-generic casper lupin-casper`

### Optional: Add update and backport repos

If you would like to include update and backport repos to ensure that the packages are the most recent, you can do so by adding the **precise-updates** and **precise-backports** repos by editing `/etc/apt/sources.list`:

* `echo 'deb http://archive.ubuntu.com/ubuntu precise main universe multiverse' > /etc/apt/sources.list; echo 'deb http://archive.ubuntu.com/ubuntu precise-updates main universe multiverse' >> /etc/apt/sources.list; echo 'deb http://archive.ubuntu.com/ubuntu precise-backports main universe multiverse' >> /etc/apt/sources.list; apt-get update`
* `apt-get upgrade`

### 5. Install additional packages

* `apt-get install --no-install-recommends xinit openbox obconf xserver-xorg x11-xserver-utils xterm network-manager-gnome plymouth-x11 plymouth-label plymouth-theme-ubuntu-logo`
* `apt-get install --no-install-recommends pcmanfm chromium-browser gtk-theme-switch shimmer-themes gnome-icon-theme gnome-brave-icon-theme dmz-cursor-theme maximus gpicview leafpad lxappearance lxmenu-data lxrandr lxterminal nitrogen ttf-ubuntu-font-family alsamixergui pm-utils libnotify-bin notify-osd notify-osd-icons`
* `apt-get install --no-install-recommends time hdparm openssh-client libglib-perl libgtk2-perl libxml-simple-perl gtk2-engines-pixbuf beep rsync smartmontools gnome-disk-utility policykit-1-gnome policykit-desktop-privileges baobab gsettings-desktop-schemas gparted gksu lshw-gtk testdisk usb-creator-gtk wodim curlftpfs nmap cifs-utils libnotify-bin cryptsetup reiserfsprogs dosfstools ntfsprogs ntfs-3g hfsutils reiser4progs jfsutils smbclient wget system-config-lvm fsarchiver partclone`
* `ln -s /usr/bin/pcmanfm /usr/bin/nautilus`
* `gconftool-2 --set /apps/maximus/undecorate --type BOOL false`
* `rm /usr/bin/{rpcclient,smbcacls,smbclient,smbcquotas,smbget,smbspool,smbtar}`
* `rm /usr/share/icons/*/icon-theme.cache; rm -rf /usr/share/doc; rm -rf /usr/share/man`

These steps will download about 145MB. These packages form the bulk of the live CD's capabilities. It is possible that some of these packages are not required, or that many of the files they install can be safely removed before creating the filesystem image in order to save space.

### 6. Extract the changed files archive

At this point, the snapshot archive containing all new or modified files should be extracted into the existing directory. Snapshot source files are available from the [https://sourceforge.net/projects/redobackup/files/src/ SourceForge download page].

* Open another shell terminal and change to the original working directory (the one that contains the "livecd" folder)
* Enter the following commands. **Again, you must open a NEW terminal window before you execute these commands! See above.**
  * `sudo su (Make sure you are in the correct directory, see above)`
  * `wget INSERT_PATH_TO_LATEST_SNAPSHOT_SRC_FILE_FROM_SOURCEFORGE_HERE`
  * `tar zxvf snapshot-X.X.X.tar`
  * `cp adeskbar*.deb livecd/chroot/`
* Close this terminal
* Return to the chroot terminal, and execute:
  * `cd /`
  * `dpkg -i adeskbar*.deb`
  * `apt-get --yes -f install`
  * `apt-get install python-wnck python-pyinotify python-alsaaudio python-vte python-xlib`
  * `rm -rf *.deb`

### 7. Update plymouth splash screen

The previous step will have extracted the files that will rebrand the plymouth splash screen, but now the image must be updated and you must select the theme to use, e.g. "redo-logo auto", which is probably option 0:
* `update-alternatives --install /lib/plymouth/themes/default.plymouth default.plymouth /lib/plymouth/themes/redo-logo/redo-logo.plymouth 100`
* `update-alternatives --config default.plymouth`
* `update-initramfs -u`

### 8. Clean up and exit the chroot

* `apt-get install localepurge`
* Select the following locales to keep:
  * `en`
  * `en_US`
  * `en_US.UTF-8`
* `localepurge`
* Remove upgraded or old linux kernels if present:
  * `ls /boot/vmlinuz-3.2.**-**-generic > list.txt; sum=$(cat list.txt | grep '[^ ]' | wc -l)`
  * `if [ $sum -gt 1 ]; then`
  * `dpkg -l 'linux-*' | sed '/^ii/!d;/'"$(uname -r | sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d' | xargs sudo apt-get -y purge`
  * `fi`
  * `rm list.txt`
* `rm /var/lib/dbus/machine-id; rm /sbin/initctl; dpkg-divert --rename --remove /sbin/initctl; apt-get clean; rm -rf /tmp/*; rm /etc/resolv.conf; rm -rf /var/lib/apt/lists/????????*; umount -lf /proc; umount -lf /sys; umount -lf /dev/pts; exit;`
* `umount -lf chroot/dev/; rm chroot/root/.bash_history`

### 9. Prepare to build the image
* `apt-get install syslinux squashfs-tools genisoimage; mkdir -p image/casper image/isolinux image/install; cp chroot/boot/vmlinuz-3.2.*-generic image/casper/vmlinuz; cp chroot/boot/initrd.img-3.2.*-generic image/casper/initrd.lz; cp chroot/usr/lib/syslinux/vesamenu.c32 chroot/usr/lib/syslinux/isolinux.bin image/isolinux/; cp /boot/memtest86+.bin image/install/memtest`

### 10. Create manifest

These steps make the image conform to Ubuntu remix specs, which probably helps with creating startup disks:

* `chroot chroot dpkg-query -W --showformat='${Package} ${Version}\n' | tee image/casper/filesystem.manifest; cp -v image/casper/filesystem.manifest image/casper/filesystem.manifest-desktop; REMOVE='ubiquity ubiquity-frontend-gtk ubiquity-frontend-kde casper lupin-casper live-initramfs user-setup discover xresprobe os-prober libdebian-installer4'; for i in $REMOVE`
* At the interactive prompt, you will need to enter these three lines:
  * `do`
  * `sed -i "/${i}/d" image/casper/filesystem.manifest-desktop`
  * `done`
* vi `image/README.diskdefines` and add these lines:
```
#define DISKNAME Redo Backup
#define TYPE binary
#define TYPEbinary 1
#define ARCH i386
#define ARCHi386 1
#define DISKNUM 1
#define DISKNUM1 1
#define TOTALNUM 0
#define TOTALNUM0 1
```
* `touch image/ubuntu; mkdir image/.disk; cd image/.disk; touch base_installable; echo "full_cd/single" > cd_type; echo "Ubuntu Remix" > info; echo "http://redobackup.org" > release_notes_url; cd ../..`

### 11. Create ISO image
* `rm -rf image/casper/filesystem.squashfs redo.iso; mksquashfs chroot image/casper/filesystem.squashfs -e boot; printf $(sudo du -sx --block-size=1 chroot | cut -f1) > image/casper/filesystem.size; (cd image && find . -type f -print0 | xargs -0 md5sum | grep -v "./md5sum.txt" > md5sum.txt); cd image; mkisofs -r -V "Redo Backup" -cache-inodes -J -l -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o ../redo.iso . ; cd ..`

The resulting ISO weighs in around 250MB. You can now boot from the ISO file in a VM, burn the image to a CD-ROM, or create a bootable USB with Unetbootin.

### 12. Correct the MD5 checksums

Something strange happens to the file `isolinux/isolinux.bin`, and the boot menu option to check the CD for defects will fail with an error unless you rerun the same command from the above step again.

### Errata

* The attempt to force the clock to show the local time in `/etc/default/rcS` by making `UTC=no` does not work. Instead, we run `hwclock -s --localtime --noadjfile` to set the time to the local hardware clock, then disable /etc/network/if-up.d/ntpdate from updating it, which should work for most users.
* Installing the `pcmanfm` package without `--no-install-recommends` will fix mounting network drives, the trash can, and automounting of inserted media and encrypted volumes -- however, doing so makes the hard drives always appear busy, which makes repartitioning and formatting them impossible. If you have information on how to retain these features, yet still be able to partition a drive and have the partition table re-read, please let us know.
* To edit the menus to your liking, you can `apt-get install --no-install-recommends alacarte gnome-panel` for a simple menu editor.
* To assist in development of the Redo Backup perl scripts, you can `apt-get install --no-install-recommends geany glade-gtk2` to develop and test inside the live CD environment.
* To beep when a long procedure completes, first `modprobe pcspkr` to load the module, and then run `beep -f 1200 -n -f 1800 -n -f 2400` or similar.
