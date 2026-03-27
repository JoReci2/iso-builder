# Create the custom iso

## Automated build with Ansible (recommended)

The included `playbook.yml` automates the entire ISO build process and
automatically detects the latest stable AlmaLinux 9 release.

### Prerequisites

- Ansible 2.9 or later
- A Linux host with `sudo` privileges (for mounting the ISO and installing packages)

### Run the playbook

```bash
ansible-playbook playbook.yml
```

The playbook will:
1. Install the required build tools (`genisoimage`, `syslinux`, `isomd5sum`, `xorriso`)
2. Detect the latest stable AlmaLinux 9.x version from the official repository
3. Download the AlmaLinux minimal ISO
4. Extract the ISO contents and overlay the custom configuration files
5. Build the final custom ISO using `mkisofs`, `isohybrid`, and `implantisomd5`

The finished ISO is written to `/tmp/iso-builder/AlmaLinux-<version>-x86_64-custom.iso`.

### Variables

You can override the following variables on the command line:

| Variable | Default | Description |
|---|---|---|
| `almalinux_major_version` | `9` | Major version series to build for |
| `arch` | `x86_64` | Target CPU architecture |
| `iso_type` | `minimal` | Source ISO type (`minimal`, `dvd`, `boot`) |
| `work_dir` | `/tmp/iso-builder` | Working / output directory |

Example – build for a specific major version:

```bash
ansible-playbook playbook.yml -e almalinux_major_version=9
```

---

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