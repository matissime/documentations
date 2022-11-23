# Adding disks to Proxmox with LVM

Proxmox LVM Expansion, adding additional disks to your Proxmox host for VM storage.

First thing, load into the Proxmox server terminal, either with the keyboard and mouse or via the Web GUI’s Shell option. You’ll want to be root.

Next, use ***fdisk -l*** to see what disk you’ll be attaching, mine looked something like this:

```bash
Disk /dev/sda: 7.36 TiB, 8096650887168 bytes, 15813771264 sectors
Disk model: RAID 5/6 SAS 6G 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: C4342EAE-7A1F-45AF-8EAD-335D4819B8B9

Device     Start         End     Sectors  Size Type
/dev/sda1   2048 15813771230 15813769183  7.4T Linux filesystem


Disk /dev/sdb: 135.97 GiB, 145999527936 bytes, 285155328 sectors
Disk model: Logical Volume  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: A65992ED-8795-43D9-AAB7-DD74AE2F4668

Device       Start       End   Sectors   Size Type
/dev/sdb1       34      2047      201[ **kenmoini.com** ] : https://kenmoini.com/post/2018/10/quick-n-dirty-adding-disks-to-proxmox/ "Quick n’ Dirty – Adding disks to Proxmox with LVM"4  1007K BIOS boot
/dev/sdb2     2048   1050623   1048576   512M EFI System
/dev/sdb3  1050624 285155294 284104671 135.5G Linux LVM


Disk /dev/mapper/pve-swap: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/pve-root: 33.75 GiB, 36238786560 bytes, 70778880 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

What I’m looking for is that /dev/sda device. Let’s work with that.



Next, we’ll initialize the partition table, let’s use cfdisk for that…

```bash
cfdisk /dev/sda
```



Navigate around…

```
> New -> Primary -> Specify size
> Write
> Quit
```



Great, next let’s create a Physical Volume from that partition. It’ll ask if you want to wipe, press ***Y***…

```bash
pvcreate /dev/sda1
```



Next we’ll extend the ***pve*** Volume Group with the new Physical Volume…

```bash
vgextend pve /dev/sda1
```



We’re almost there, next let’s extend the logical volume for the PVE Data mapper…we’re increasing it by 7.4TB, you can find that size by seeing how much is available with the vgs command

```bash
lvextend /dev/pve/data -L +7.4t
```



And that’s it! now if we jump into Proxmox and check the Storage across the Datacenter we can see it’s increased! Or we can run the command…

```bash
lvdisplay
```



------

*References:*

*[ **kenmoini.com** ] : https://kenmoini.com/post/2018/10/quick-n-dirty-adding-disks-to-proxmox/ "Quick n’ Dirty – Adding disks to Proxmox with LVM"*
