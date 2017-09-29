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
- Run [badblocks](https://wiki.archlinux.org/index.php/badblocks)
  - `badblocks -c 65536 -b 4096 -vs /dev/sdX`

  
## Format the new disk
- For your new disks, format them with ext4
  - `mkfs.ext4 -L drive# -j /dev/sdX` - replace `drive#` and `sdX` with drive number and device id respectively. 
  - When prompted, proceed to format the entire device using 'y'
- Tell the kernel to re-read the partition table
  - `sfdisk -R`

## Mount the disk on startup
- Identify the hard drive to mount and copy the UUID from the output that corresponds to the hard drive (i.e. sdb)
  - `ls -l /dev/disk/by-uuid/`
- Add the following to the end of /etc/fstab, replacing the UUID as captured in the previous step
  - `nano /etc/fstab`
```
#
# /etc/fstab
# Created by anaconda on Sat Nov  9 01:46:39 2013
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=1ebbf241-528c-465e-889f-acc15400dd8c /                       ext4    defaults        1 1
UUID=087b15a5-c3ca-4615-b6ee-bf5f399a803e /boot                   ext4    defaults        1 2
UUID=75346b8e-b162-458c-b0e9-a8d48ec2bc82 swap                    swap    defaults        0 0
UUID=ad85eeb9-18f0-4b85-9bfa-b88a5d1489b3 swap                    swap    defaults        0 0
UUID=547b073d-e591-4913-b4fb-7c5084353979 /var/hda/files/drives/drive1 ext4 defaults 1 2
```
- Test the modified fstab file
  - `mount -a` - If all goes well, there should not be any output. If there are errors, stop and diagnose the problem.





## Add the disk to MergerFS

## (Optional) RSync over any data from previous disk. 


## Resources
- https://wiki.amahi.org/index.php/Adding_a_second_hard_drive_to_your_HDA
- https://askubuntu.com/questions/334022/mount-error-special-device-does-not-exist
