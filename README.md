# Create the custom iso

## Extract the source

```bash
mkdir /mnt/almalinux
mount -o loop AlmaLinux-9.5-x86_64-minimal.ios /mnt/almalinux
cp -rv /mnt/almalinux .
umount /mnt/almalinux
```

## Copy the files in the destination

```bash
cp iso-builder/EFI/BOOT/grub.cfg almalinux/EFI/BOOT/grub.cfg
cp iso-builder/isolinux/isolinux.cfg almalinux/isolinux/isolinux.cfg
cp -r iso-builder/kickstart almalinux
```

## Build the final iso

```bash
mkisofs -o AlmaLinux-9.5-x86_64-custom.iso \
    -b isolinux/isolinux.bin -J -R -l \
    -c isolinux/boot.cat \
    -no-emul-boot \
    -boot-load-size 4 \
    -boot-info-table \
    -eltorito-alt-boot \
    -e images/efiboot.img \
    -no-emul-boot \
    -graft-points \
    -V "AlmaLinux-9.5-x86_64-custom" almalinux

isohybrid --uefi AlmaLinux-9.5-x86_64-custom.iso
implantisomd5 AlmaLinux-9.5-x86_64-custom.iso
```