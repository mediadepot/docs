This is the WIP design documentation for mediadepot v2.
For archived v1 docs, check the [`v1` branch](https://github.com/mediadepot/docs/tree/v1)

# Goal
The overall goal for this project is to automate the setup and configuration of an home server.

# Requirements

- The server will be self hosted, with only one node (if you need a multihosted media server, this wont work for you)
- The server will be running headless (no monitor is required)
- The server will be running a minimal OS/hypervisor. This is to limit the amount of OS maintenance required, and ensure
that all software is run in a maintainable & isolated way.
- The server will be using JBOD disk storage (allowing you to aggregate and transparently interact with multiple physical disks as a single volume)
    - Redundancy is supported but not a requirement.

# Design

The configured server will run the following software. Check the [DESIGN.md](DESIGN.md) document for reasoning and dismissed alternatives.

## OS
CoreOS was chosen as the Operating System for MediaDepot servers. A container-focused OS that's designed for painless management and
very little overhead. It assumes that user workloads and applications are run inside docker containers, leaving the OS clean and minimal.

See [DESIGN.md](DESIGN.md) for more information

## `MediaDepot` Services
During installation the MediaDepot configuration engine ([mediadepot/ignition](https://www.github.com/mediadepot/ignition))
will install a handful of Systemd services (mostly running docker containers). These are **required** services,
and provide a foundation that all user Docker containers can build ontop of.

- [MergerFS](https://github.com/trapexit/mergerfs) - JBOD disk storage (allowing you to aggregate and transparently interact with multiple physical disks as a single volume)
- [DNSMasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) - Runs a tiny DNS server, ensuring that all ``*.depot.lan` requests resolve to the server.
- [NetData](https://github.com/netdata/netdata) - Runs a efficient performance monitoring application that can be used to view server utilization remotely
- [Samba](https://www.samba.org/) - Runs a Samba (SMB) server (which is compatible with all major OS's), with automatically created network shares.
- [SmartMonTools](https://www.smartmontools.org/) - S.M.A.R.T disk monitoring to ensure that all storage hard drives are working correctly.
- [Traefik](https://traefik.io/) - Reverse Proxy, used to automatically route `*.depot.lan` requests to applications running in containers.
- [Portainer](https://www.portainer.io/) - UI for Docker daemon, also supports templates

See [DESIGN.md](DESIGN.md) for more information

## Storage Locations

### Storage Drives
MediaDepot will mount your hard drives to subfolders in the `/mnt` directory. These subfolders will be numbered and prefixed with `drive`.
You will not have to create these folders, or interact with these mounted drives directly. The `mergerfs` filesystem will transparently proxy
file operations to the correct disks.

```
/mnt/
├── drive1/ - mount points for all hard drives
├── drive2/
├── drive3/
```

## MergerFS Drive
The `mergerfs` JBOD volume will be mounted to the `/media/storage` folder. It is the primary file system used by all applications
and containers on MediaDepot.

```
/media/
├── storage/ - mergerfs mount point
```

## Temp Storage
To limit usage of storage drives and decrease thrashing across multiple disks, high IO disk operations are targeted to the
OS disk. We create a `/media/temp` directory to act as a cache directory for all these high IO operations.

The `/media` directory is actually a `tmpfs` volume on CoreOS, which means it's stored on a RAMDisk and is destroyed during restarts.
To solve this, we will bind mount the `/mnt/temp` to `/media/temp`.

```
/mnt/
├── temp/ - cache directory for high IO operations

/media/
├── temp/ - mount point for cache directory
```

## Application Data
Application settings and data is stored in the `/opt/mediadepot/apps/` directory of the OS disk. Subfolders of this directory are mounted
into user created containers.

```
/opt/mediadepot/
├── apps/ - user application settings and data
│   ├── sickrage/
│   ├── plex/
│   └── couchpotato/
```

This directory can be easily backed up to cloud storage of your choice using the [`mediadepot/rclone`](https://www.github.com/mediadepot/docker-rclone) container.

## Media Storage Structures



## Containers & Portainer Templates


# Installation

## CoreOS Ignition








---

> ALL DOCUMENTATION BELOW HERE IS OLD AND NEEDS TO BE RE-WRITTEN AND MOVED ABOVE.


# Planning
This is an organization to group all the configuration and docker images required to setup a self-hosted media server with the following capabilities:

- JBOD disk storage (allowing you to aggregate and transparently interact with multiple physical disks as a single volume)
- Media server applications such as plex, sickbeard, couchpotato, etc to manage and view media
- Utility applications such as ajenti, openvpn, conky, btsync, bittorrent.
- Notifications system (so that you are notified whenever any service stops or starts, and when media is added)
- Easy system OS updates - via a minimal OS (CoreOS or Atomic Host)

# Assumptions
Here are a couple of assumptions we are making:
- Drive storage will be configured using a JBOD array that will be mounted on the host, and can be shared with the docker containers
- Pushover will be used to send remote notifications
- Assumes that all drives have been formatted and mounted 
- **(NEW)** Dockerized applications will be run using [LinuxServer](https://github.com/linuxserver) created images where possible, leveraging open source community
- **(NEW)** Dockerized applications will store their configuration on the file system, to ensure that it can persist across container destruction
- **(NEW)** Minimal OS which primarily runs docker daemon.
- **(NEW)** Container management is done via Rancher v2


# Host Applications
The following software will run on the host:
- JBOD/Union FS (Mergerfs)
- SSH daemon
- Samba
- Docker
- SMART disk status - https://www.hdsentinel.com/hard_disk_sentinel_linux.php or similar
- **(NEW)** SnapRAID for filesystem checksum


# Docker Containers
The following software can be run via docker containers:
- **(NEW)** **(REQUIRED)** Dynamic DNS updater script/DuckDNS
- **(NEW)** **(REQUIRED)** Rancher v2
- **(NEW)** Healthchecks/Healthchecks container for Cron status tracking https://hub.docker.com/r/galexrt/healthchecks
- **(NEW)** Backup via Rclone
- Vault or other credential/secret management system
- Ajenti? (does this make sense, or should it just be run on the Host)
- [Couchpotato](https://github.com/mediadepot/docker-couchpotato)
- [Deluge](https://github.com/mediadepot/docker-deluge)/Hadouken/ruTorrent/[rTorrent](https://github.com/mediadepot/docker-rtorrent) (torrent downloaders)
  - Supports multiple watch directories
  - Supports multiple/dynamic download directories
  - Supports multiple/dynamic completed directories
  - Supports user/password authentication
  - Supports auto-labeling
  - Supports scheduling/QoS
  - Can work in docker container
  - Auto cleanup?
- [Headphones](https://github.com/mediadepot/docker-headphones)
- [Plex](https://github.com/mediadepot/docker-plex)/Emby
- [Plex-Requests](https://github.com/mediadepot/docker-plexrequests)/Ombi
- [PlexPy](https://github.com/mediadepot/docker-plexpy)/Tautulli
- [Sickrage](https://github.com/mediadepot/docker-sickrage)/Sickbeard
- [Headphones](https://github.com/mediadepot/docker-headphones)
- Nginx/HAProxy router
- Log viewer web app /[splunk](http://www.splunk.com/en_us/products/splunk-enterprise/free-vs-enterprise.html)/fluentd webui/ loggly aggregator/Graylog/rtail
- [BTSync](https://github.com/mediadepot/docker-btsync)/SyncThing/Seafile/Pydio/Nextcloud
- [FileRun](https://github.com/mediadepot/docker-filerun) - file explorer
- duplicati - Backup service
- Update checker service
- Lychee/Photato/Koken.me/chevereto
- [Guacamole (VNC)](https://github.com/mediadepot/docker-guacamole)
- [LazyLibrarian](https://github.com/mediadepot/docker-lazylibrarian)
- OpenVPN_AS/pritunl
- StremIO
- Status Page/Dashboard - Dashing/Atlasboard
- Sandstorm
- openbazaar
- [compactd](https://github.com/compactd/compactd)/AirSonic/SubSonic/[Madsonic](https://github.com/mediadepot/docker-madsonic)/Sonarr/koel for Music/Mopidy/sonerezh/cloudtunes
- Huginn/Bip Automation
- SqlPad
- visallo
- dillinger
- [jDownloader](https://hub.docker.com/r/aptalca/docker-jdownloader2/)
- alltube
- bookstack
- tagspaces
- dashboard/[muximux](https://github.com/mescon/Muximux)
- monica - personal relationship manager
- paperless/ambar/mayan EDMS - document archival system + search
- [lidarr](https://github.com/lidarr/Lidarr)
- [klaxon](https://github.com/themarshallproject/klaxon) website change tracking
- Ideas for other containers - https://github.com/DFabric/DPlatform-ShellCore https://github.com/Kickball/awesome-selfhosted#read-it-later-lists

# Container Configuration
- All containers will have configuration service to dynamically generate config files:
  - https://github.com/jwilder/docker-gen
  - https://github.com/jwilder/dockerize
  - https://github.com/hashicorp/consul-template
  - https://github.com/markround/tiller
  - https://github.com/gliderlabs/registrator

# Installation Script
An `install.sh` script will be created that can be run using:
  
    wget -O - http://example.com/install.sh | sudo sh
  
The installer script will be [wrapped in {} ](https://www.reddit.com/r/programming/comments/1pnkxs/dont_pipe_to_your_shell/cd4bu30) to verify download.

The installation script will setup Chef-client by installing ChefDK, run berks install on the cookbook to download all dependencies and then copy the cookbook environment files to the correct location. It will then bootstrap the chef-client run using chef-zero.

Should ask the user for input, ie. run a wizard or something similar to generate the environment file with the correct secrets + config:
- default username/password
- github token key
- pushbullet token key
- which ssh keys public keys to add
- github gist id
- hostname -- used for generating the dns names for each service
- duckdns token + domain
- greyhole disk drives to include. 

On second run, it should allow using existing configuration, or allow the user to change any previously specified options.

# Depot Chef Cookbook
- Should attempt to retrieve data from persistent data location (github gist) to use when configuring applications
- Should install all the above listed Host Application Software
- Should create a root SSL CA certificate and client cerificates for all client applications that require them. 
- Should install the Rancher docker container using chef-rancher cookbook (that we may have to create)
- Define a docker-compose.yml file depending on the options specified in the environment file
- Run docker-compose (rancher-compose)
- Create/Update the persistent data file in github gist



# Persistent Data
- Sickbeard Tv show names and ids will be saved
- Couchpotato Movie names and ids will be saved
- Hosts file entry template

# Issues
- how to handle mounting drives. Should they be mounted via chef. If  yes, how do we configure all the various options for each drive, if no, how do we select all the mounted drives to use with greyhole. 

# Future Ideas
- Create a rancher catalog repository to include base templates that are currently not supported. Allows for creating a rancher based application store. 
  - https://github.com/rancher/rancher/wiki/Setup:-Settings
  - http://localhost:9090/v1/settings
    - service.package.catalog.url
    - catalog.url

# Service Discovery/Automatic SubDomain Assignment/Load Balancing
- http://docs.rancher.com/rancher/rancher-ui/applications/stacks/adding-balancers/#advanced-routing-options
- https://github.com/rancher/rancher/issues/2367
- http://rancher.com/the-magical-moment-when-container-load-balancing-meets-service-discovery/
- http://rancher.com/load-balancing-support-for-docker-in-rancher/
- http://rancher.com/load-balancing-support-for-docker-in-rancher/
- http://docs.rancher.com/rancher/labels/
- http://rancher.com/virtual-host-routing-using-rancher-load-balancer/
- http://docs.rancher.com/rancher/rancher-compose/
- http://docs.rancher.com/rancher/rancher-compose/rancher-services/#advanced-load-balancing-(l7)



# Logging
- http://blog.treasuredata.com/blog/2015/08/03/5-use-cases-docker-fluentd/

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

# References
- http://www.cyberciti.biz/tips/spice-up-your-unix-linux-shell-scripts.html
- https://www.reddit.com/r/programming/comments/1pnkxs/dont_pipe_to_your_shell/cd4bu30
- https://github.com/jwilder/docker-gen
- https://github.com/hashicorp/consul-template
- https://github.com/markround/tiller
- https://github.com/gliderlabs/registrator
- https://en.wikibooks.org/wiki/Bash_Shell_Scripting/Whiptail
- http://xmodulo.com/create-dialog-boxes-interactive-shell-script.html
- http://tldp.org/LDP/abs/html/here-docs.html
- http://www.faqforge.com/linux/get-the-disk-health-status-with-smart-monitor-tools-on-debian-and-ubuntu-linux/
- https://github.com/veggiemonk/awesome-docker
- https://github.com/CenturyLinkLabs/watchtower
- https://github.com/phusion/baseimage-docker
- http://gliderlabs.viewdocs.io/docker-alpine/
- https://muffinresearch.co.uk/linux-changing-uids-and-gids-for-user/
- https://github.com/firecat53/dockerfiles
- https://github.com/Kickball/awesome-selfhosted
- https://github.com/docker/docker/pull/11253
- https://github.com/docker/docker/pull/12648
- https://github.com/docker/docker/issues/7198
- https://docs.docker.com/engine/reference/api/docker_remote_api_v1.21/#monitor-docker-s-events
- https://help.ubuntu.com/community/Smartmontools
- https://github.com/jtimberman/smartmontools-cookbook/blob/master/recipes/default.rb
- https://www.smartmontools.org/browser/trunk/smartmontools/smartctl.8.in

