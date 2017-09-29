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

## Verify the disk health
- Run [badblocks](https://wiki.archlinux.org/index.php/badblocks)
  - `badblocks -b 4096 -vs /dev/sdX`

  
## Format the new disk
- For your new disks, format them with ext4
  - `mkfs.ext4 -L drive# -j /dev/sdX` - replace `drive#` and `sdX` with drive number and device id respectively. 
  - When prompted, proceed to format the entire device using 'y'


## Mount the disk on startup


## Add the disk to MergerFS

## (Optional) RSync over any data from previous disk. 
