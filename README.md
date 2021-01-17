# `isorespin.sh`

This version of the isorespin script is specifically hacked together to unpack and repack GalliumOS images, an Ubuntu flavored distro with Chrome hardware patches. 

Extract MBR/EFI information from existing ISO:
```
mbr=isorespin/galliumos-3.1-apollolake.mbr
efi=isorespin/galliumos-3.1-apollolake.efi
dd if="$orig" bs=1 count=446 of="$mbr"
skip=$(/sbin/fdisk -l "$orig" | fgrep '.iso2 ' | awk '{print $2}')
skip=$(/sbin/fdisk -l "$orig" | grep '.iso2 ' | awk '{print $2}')
size=$(/sbin/fdisk -l "$orig" | fgrep '.iso2 ' | awk '{print $4}')
dd if="$orig" bs=512 skip="$skip" count="$size" of="$efi"\n
```

Run the script with:
```
sudo ./isorespin.sh -i galliumos-3.1-apollolake.iso -k v5.10 -w . --apollo
```

The script will pause at this point after creating the squashfs. 
```
...
161970+0 records in
161970+0 records out
82928640 bytes (83 MB, 79 MiB) copied, 1.41604 s, 58.6 MB/s
GALLIUMOS
Press enter to continue...
```

Move into the iso-directory-structure directory and build the ISO manually with the following xorriso command.

```
sam@aleppo:pts/3->/home/sam/isorespin/isorespin/iso-directory-structure (0)
·> ls
apt  boot  boot.catalog  casper  EFI  efi.img  mach_kernel  md5sum.txt  preseed  README.isorespin  System
·> xorriso -as mkisofs \
  -r -V GALLIUMOS -J -joliet-long -l \
  -iso-level 3 \
  -partition_offset 16 \
  --grub2-mbr ~/isorespin/galliumos-3.1-apollolake.mbr \
  --mbr-force-bootable \
  -append_partition 2 0xEF ~/isorespin/galliumos-3.1-apollolake.efi \
  -appended_part_as_gpt \
  -c /boot.catalog \
  -b /boot/grub/i386-pc/eltorito.img \
    -no-emul-boot -boot-load-size 4 -boot-info-table --grub2-boot-info \
  -eltorito-alt-boot \
  -e '--interval:appended_partition_2:all::' \
    -no-emul-boot \
  -o ../../iso.iso \
  .                     
```

The resulting ISO can be built flashed onto a drive and booted as you would expect. Uname -r will return 5.10.0+ generic.

See the [original isorespin README here](https://github.com/kenorb-contrib/isorespin/blob/master/README.md).
