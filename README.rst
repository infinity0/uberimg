This package contains GRUB scripts that let you boot ISO images contained on a live medium (e.g. a USB stick). This means that you can put multiple ISO images on a single medium and boot any of them as needed.

This is different from the usual method, where you must write the *contents* of the ISO image to the medium, so that you need a separate medium for each distro you want to use.

`Super Grub2 Disk <http://www.supergrubdisk.org/>`_ has this ability too; in fact I gained a lot of knowledge needed to write this script by talking with those guys. If you just want to boot from ISO files, you might want to use that. However, the scripts here give you a bit more flexiblity, if you want to do other things with the USB disk too.

This requires GRUB2 version 1.99 or later.

Instructions
============

1.  Either (a) insert your physical device (then umount all partitions, if your system does auto-mounting), or (b) create a virtual disk image file using ``./imgctl create``
2.  Create any partitions as necessary. I suggest a separate small boot partition and a much larger data partition.

	a.  For EFI booting, your boot partition **MUST** be FAT32-formatted; 100MB should be enough. If you want to also support legacy BIOS boot mode on the same device, then you'll need a second "boot" partition of 2MB with the ``bios_grub`` partition flag.

3.  Install the bootloader:

	1.  Mount the boot partition onto some ``$MNT`` directory. Let ``$BOOT`` be the mounted boot directory, which should either be ``$MNT/`` or ``$MNT/boot/``.
	2.  Run ``./imgctl install $BOOT``. If it doesn't detect the device correctly, cancel it with Ctrl-C and re-run ``./imgctl install $BOOT /dev/$DEVICE``.

		- You can pass extra options to ``grub-install`` via the ``GRUB_OPTS`` envvar.
		- We install the x86_64-efi target by default. You can change this by passing e.g. ``GRUB_TARGET=i386-pc`` as an envvar. You can install multiple targets onto the same device, but your device layout must be compatible with all the targets, see the notes in step (2).

	3.  Unmount the boot partition from ``$MNT``.

4.  Install your data:

	1.  Mount the data partition onto some ``$MNT`` directory. Let ``$DATA`` be the mounted data directory, which should either be ``$MNT/`` or ``$MNT/boot/``.
	2.  Copy your ISOs to ``$DATA/img/``
	3.  (Unnecessary for most people) Copy any overrides to ``$DATA/linux/`` and ``$DATA/initrd/``

		- The override for ``XXXX.iso`` should be named ``XXXX``, e.g. ``XXXX.vmlinuz`` or ``XXXX.initrd.gz``
		- For example, Backtrack 5 needs a newer version of casper-generated ``/scripts/`` for its initrd.

	4.  Umount the data partition from ``$MNT``.

5.  Use ``./imgctl test <image file|device>`` to test it with QEMU. (Of course, install qemu if you don't already have it.)

If you get *"Unsupported ISO type"* for any of your ISOs, file me a bug! If this script can be modified to support it, I will do so; otherwise I will tell you to file a bug to the upstream ISO developers.

You can also try to fix the bug yourself by tweaking ``isodetect.cfg``, or playing around with overrides in step 4.3. above.

Known bugs
==========

Sometimes the installation process doesn't work if you install over an existing installation, e.g. the result goes into a boot loop. To fix:

1. Backup any files you manually edited in boot/grub/*
2. Wipe the boot partition, and re-apply {esp, boot} partition flags.
3. Wipe the bios_grub partition, and re-apply {bios_grub} partition flags.
4. Install it again, and test it with QEMU.
5. It should now work. Restore your files from step 1.
