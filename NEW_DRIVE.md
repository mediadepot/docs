# Add a new Disk

## Install the new disk
- Shutdown the server safely
- Remove any disks that need to be removed
- Install any new disks
- Start up the server

## Determine the disk information
- Determine what disks are currently mounted
  - `df -h`
- Determine the device id's for new disks
  - `ls -l /dev/disk/by-id`
  - `ls -l /dev/disk/by-label`
- Verify the device id's match your new disks using disk info
  -  `smartctl -a /dev/sdX` - replace `sdX` with your device id. 

## (Optional) Verify the disk health
- This step can take hours, and is not really required as Mediadepot has a cronjob that tests for bad blocks and runs S.M.A.R.T tests continuously. 
- Run [badblocks](https://wiki.archlinux.org/index.php/badblocks)
  - `badblocks -c 65536 -b 4096 -vs /dev/sdX`

  
## Format the new disk
- For your new disks, format them with ext4
  - `mkfs.ext4 -L "8.0TB-WD-3" -j /dev/sdX` - replace `8.0TB-WD-3` with disk size, manufacturer and slot#. replace `sdX` with device id. 
  - When prompted, proceed to format the entire device using 'y'
- Tell the kernel to re-read the partition table
  - `sfdisk -R`

## Mount the disk on startup
- Identify the hard drive to mount and copy the UUID from the output that corresponds to the hard drive (i.e. sdb)
  - `ls -l /dev/disk/by-uuid/`
- Add the UUID to the /etc/fstab, replacing the UUID as captured in the previous step
  - `nano /etc/fstab`
- If this is a new disk, make sure you also add it's mount location to the MergerFS line. 
```
#
# /etc/fstab
# Created by anaconda on Sat Nov  9 01:46:39 2013
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=1ebbf241-528c-465e-889f-acc15400dd8c /                       ext4    defaults        1 1
UUID=75346b8e-b162-458c-b0e9-a8d48ec2bc82 swap                    swap    defaults        0 0
UUID=ad85eeb9-18f0-4b85-9bfa-b88a5d1489b3 /mnt/drive1             ext4    defaults        0 2
UUID=547b073d-e591-4913-b4fb-7c5084353979 /mnt/drive2             ext4    defaults        0 2

/mnt/drive1:/mnt/drive2:/mnt/drive3 /media/storage fuse.mergerfs direct_io,defaults,allow_other,minfreespace=50G,fsname=mergerfs 0 2

```
- Test the modified fstab file
  - `mount -a` - If all goes well, there should not be any output. If there are errors, stop and diagnose the problem.
  - a response of `fuse: mountpoint is not empty` can be ignored. its a message from MergerFS. 

## Add the disk to MergerFS
- Temporarily add the new disk mount path to the MergerFS system. 
  - `xattr -w user.mergerfs.srcmounts +/mnt/driveX /media/storage/.mergerfs` - replace `driveX` with mount location
- Rebalance the MergerFS disks
  - `mergerfs.balance /media/storage`

## (Optional) RSync over any data from previous disk. 

This command can be used to synchronize a folder, and also resume copying when it's aborted half way. The command to copy one disk is:

`rsync -avxHW --numeric-ids --info=progress2 /mnt/disk_source/ /mnt/disk3/`

The options are:

```
-a  : all files, with permissions, etc..
-v  : verbose, mention files
-x  : stay on one file system
-H  : preserve hard links (not included with -a)
-W  : copy files whole (without rsync algorithm)
--remove-source-files : delete source files on sync complete

# The following options are not supported in CoreOS/Flatcar
-A  : preserve ACLs/permissions (not included with -a)
-X  : preserve extended attributes (not included with -a)
--numeric-ids : don't map uid/gid values by user/group name
--info=progress2 : instead of --progress is useful for large transfers, as it gives overall progress
```
Trailing slash on `/mnt/disk_source/` is important as it means that the *content* of `disk_source` folder will be copied into the destionation folder, instead of copying the `disk_source` folder *into* the destination. 


## Resources
- https://wiki.amahi.org/index.php/Adding_a_second_hard_drive_to_your_HDA
- https://askubuntu.com/questions/334022/mount-error-special-device-does-not-exist
- https://github.com/trapexit/backup-and-recovery-howtos/blob/master/docs/recovery_(mergerfs,snapraid).md
- https://github.com/trapexit/mergerfs-tools
- https://superuser.com/questions/307541/copy-entire-file-system-hierarchy-from-one-drive-to-another
