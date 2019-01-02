This document explains all the different decisions that were made when designing MediaDepot.
It'll discuss the final choice, alternatives that were considered and any compromises that were necessary.

## OS

# Storage
## Union/Local JBOD
- mergerfs - https://www.linuxserver.io/index.php/2016/02/06/snapraid-mergerfs-docker-the-perfect-home-media-server-2016/
- MHDDFS
- AUFS
- BTRFS
- unRAID
- RAID (mdadm)
- ZFS


## Distributed File Systems
https://www.reddit.com/r/DataHoarder/comments/4ey8zj/looking_for_a_distributed_media_storage_solution/
https://www.reddit.com/r/HomeServer/comments/4ey3qw/looking_for_a_distributed_media_storage_solution/

- GlusterFS
- CephFS
- LizardFS
- MooseFS
- infinit.one