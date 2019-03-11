This is the WIP design documentation for mediadepot v2.
For archived v1 docs, check the [`v1` branch](https://github.com/mediadepot/docs/tree/v1)



[![](https://i.imgur.com/cl5ReOY.jpg)](https://imgur.com/a/VMcMtVY)

# Goal
The overall goal for this project is to automate the setup and configuration of an home server.

# Requirements

- The server will be self hosted, with only one node (if you need a multihosted media server, this wont work for you)
- The server will be running headless (no monitor is required)
- The server will be running a minimal OS/hypervisor. This is to limit the amount of OS maintenance required, and ensure
that all software is run in a maintainable & isolated way.
- The server will be using JBOD disk storage (allowing you to aggregate and transparently interact with multiple physical disks as a single volume)
    - Redundancy is supported but not a requirement.
- The server will provide a automation friendly folder structure for use by media managers (sickrage, couchpotato, sonar, plex, etc)
- The server will provide a monitoring solution with a web GUI.
- The server will provide a routing method to running web applications via a custom domain `*.depot.lan`
- The server will provide a method that user applications can use to notify the user when events have occured (download started, completed, media added)
- The server will provide a way to backup application configuration to a secondary location.

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

See [Containers & Portainer Templates](#)

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

## Media Storage Structures

The folder structure below may look a bit complicated, however its designed to be as automation friendly as possible, while also
being flexible enough to handle user naming schemes. The primary folders, and their uses, are as follows:

- `/media/temp/blackhole/*` - temporarily contains `.torrent` files. These files can be added manually via SMB, or automatically by apps like sickrage, couchpotato, sonarr, etc.
- `/media/temp/processing` - a cache directory used by your torent client. Temporarily holds current download files. Once complete they are moved into the correct subfolder of `/media/storage/downloads`
- `/media/storage/downloads/*` - contains completed torrent downloads. Files added here are automatically detected by media managers (sickrage, couchpotato, etc) then renamed/reorganized and moved to their
final storage directory `/media/storage/*`
- `/media/storage/*` - contains the final renamed/organized media, to be used by your media streamer of choice (Plex/Emby/etc).
All subfolders are automatically created as SMB shares


```
/media
├── storage/
│   ├── downloads/
│   │   ├── movies/
│   │   ├── music/
│   │   ├── tvshows/
│   ├── movies/
│   ├── music/
│   ├── tvshows/
├── temp/
│   ├── blackhole/
│   │   ├── movies/
│   │   ├── music/
│   │   ├── tvshows/
│   └── processing/

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

## User Containers & Portainer Templates
As mentioned earlier, MediaDepot is built ontop of CoreOS, which is primarily a minimal operating system for Docker.
All user applications should be run via Docker. While users can manually create docker containers using the `docker run` command,
we've provided a helpful registry of pre-configured and tested templates that are designed to work with MediaDepot.
You can access them by going to `http://admin.depot.lan/templates`.


<< INSERT IMAGE HERE >>

Here's a list of the maintained templates that we support.

| Application Name | Maintainer | Description |
| ------------- | ------------- | ------------- |
| [Cardigann]() | [LinuxServer](linuxserver/cardigann)  | Cardigann, a server for adding extra indexers to Sonarr, SickRage and CouchPotato |
| [Couchpotato]()  | [LinuxServer](linuxserver/couchpotato)  | CouchPotato (CP) is an automatic NZB and torrent downloader. |
| [DuckDNS]() | [LinuxServer](linuxserver/cardigann) | Duckdns is a free service which will point a DNS (sub domains of duckdns.org) to an IP of your choice. |
| [Duplicati]() | [LinuxServer](https://hub.docker.com/r/linuxserver/duplicati) | Free backup software to store encrypted backups online For Windows, macOS and Linux |
| [Droppy]() | [Silverwind](https://hub.docker.com/r/silverwind/droppy) | Droppy is a self-hosted file storage server  |
| [FileRun]() | [Afian](afian/filerun) | FileRun File Manager: access your files anywhere through self-hosted secure cloud storage |
| [Heimdall]() | [LinuxServer](linuxserver/heimdall) | Heimdall is a way to organise all those links to your most used web sites |
| [Jackett]() | [LinuxServer](linuxserver/jackett) | Jackett works as a proxy server it translates queries from apps like Sonarr etc into tracker-site-specific http queries |
| [Klaxon]() | [MediaDepot](mediadepot/klaxon) | Klaxon is a free robot that checks websites regularly so you don't have to. |
| [Mayan EDMS]() | [LinuxServer](linuxserver/jackett) | Mayan EDMS is an electronic vault for your documents. |
| [Plex Media Server]() | [LinuxServer](linuxserver/plex) | Plex organizes your video, music, and photo collections and streams them |
| [Plex Requests]() | [LinuxServer](linuxserver/plexrequests) | Simple automated way for users to request new content for Plex. |
| [Radarr]() | [LinuxServer](linuxserver/radarr) | Radarr - A fork of Sonarr to work with movies la Couchpotato. |
| **[Rclone Config Backup]()** | **[MediaDepot](mediadepot/rclone)** | **Rclone can be used to backup your application config directories to a cloud storage provider of your choice (Dropbox, GDrive, etc).** |
| [Resilio Sync]() | [LinuxServer](linuxserver/resilio-sync) | Resilio Sync (formerly BitTorrent Sync) uses the BitTorrent protocol to sync files and folders |
| **[ruTorrent]()** | **[MediaDepot](mediadepot/rutorrent)** | **ruTorrent is a quick and efficient BitTorrent client** |
| [SickRage]() | [LinuxServer](linuxserver/sickrage) | Automatic Video Library Manager for TV Shows. |
| [Sonarr]() | [LinuxServer](linuxserver/sonarr) | Sonarr (formerly NZBdrone) is a PVR for usenet and bittorrent users. |
| [Tautulli]() | [LinuxServer](linuxserver/tautulli) | A Python based monitoring and tracking tool for Plex Media Server. |
| [UrlWatch]() | [MediaDepot](mediadepot/urlwatch) | A tool for monitoring webpages for updates |
| [Watchtower]() | [Community](v2tec/watchtower) | Automatically update running Docker containers |


In addition to the pre-configured Dockerized Applications above, you may be interested in the following Self-Hosted software projects.
If there's enough interest, they may be added to our template system.

| Application Name | Maintainer | Description | Template Status |
| ------------- | ------------- | ------------- | ------------- |
| [Atlasboard](https://bitbucket.org/atlassian/atlasboard/src/master/) | [Community]() | Atlasboard is a dashboard framework written in nodejs.  |  |
| [Airsonic]() | [LinuxServer](https://hub.docker.com/r/linuxserver/airsonic) | Airsonic is a free, web-based media streamer, providing ubiquitious access to your music |  |
| [Booksonic]() | [LinuxServer](https://hub.docker.com/r/linuxserver/booksonic) | Booksonic is a server and an app for streaming your audiobooks to any pc or android phone.  |  |
| [CampaignChain](https://www.campaignchain.com/) | []() | plan, execute, monitor, optimize and integrate all your campaigns, channels, activities and milestones in one place. | |
| [Compactd](https://github.com/compactd/compactd) | [Community]() | Compactd aims to be a self-hosted remote music player in your browser, streaming from you own personal server |  |
| [Dashing]() | [Community]() | The exceptionally handsome dashboard framework. |  |
| [Deluge]() | [MediaDepot](mediadepot/deluge) | Deluge is a lightweight, Free Software, cross-platform BitTorrent client. Full Encryption; WebUI; Plugin System; | deprecated |
| [Diskover](https://shirosaidev.github.io/diskover/) | [LinuxServer](https://hub.docker.com/r/linuxserver/diskover/) | diskover is a file system crawler and disk space usage software |  |
| [Filebrowser](https://github.com/filebrowser/filebrowser) | [LinuxServer]() | Web File Browser which can be used as a middleware or standalone app.  |  |
| [FunkWhale](https://funkwhale.audio/) | []() | A modern, convivial and free music server | |
| [Healthchecks]() | [LinuxServer](https://hub.docker.com/r/lsiocommunity/healthchecks)  | healthchecks is a watchdog for your cron jobs. It's a web server that listens for pings from your cron jobs, plus a web interface. | pending |
| [Hadouken]() | []() |  |  |
| [Headphones]() | [LinuxServer](linuxserver/headphones) | headphones is an automated music downloader for NZB and Torrent, written in Python. |  |
| [Koken]() | []() | Content management and web site publishing for photographers |  |
| [KeyCloak]() | []() |  |  |
| [Lychee]() | [LinuxServer](https://hub.docker.com/r/linuxserver/lychee) | Lychee is a free photo-management tool, which runs on your server or web-space. |  |
| [LazyLibrarian]() | [LinuxServer](https://hub.docker.com/r/linuxserver/lazylibrarian) | LazyLibrarian is a program to follow authors and grab metadata for all your digital reading needs. | pending |
| [Libresonic]() | [LinuxServer](https://hub.docker.com/r/linuxserver/libresonic) | Libresonic is a free, web-based media streamer, providing ubiqutious access to your music.  |  |
| [Openvpn-As]() | [LinuxServer](https://hub.docker.com/r/linuxserver/openvpn-as) | OpenVPN Access Server is a full featured secure network tunneling VPN software solution |  |
| [Plex Media Server + CUDA]() | [MediaDepot](mediadepot/plex-cuda) | LinuxServer/plex image modifed to add support for hardware transcoding using Nvidia CUDA | requires `device` support |
| [Pritunl](https://pritunl.com/) | [Community]() | Enterprise Distributed OpenVPN and IPsec Server |  |
| [Subspace](https://github.com/subspacecloud/subspace) | []() |  A simple WireGuard VPN server GUI | |
| [Syncovery](https://www.syncovery.com/) | [Community]() | Syncovery will copy your files the way you need it |  |
| [TagSpaces](https://www.tagspaces.org) | []() | Your versatile file manager TagSpaces is an offline, open source, data manager.  It helps you to browse and organize your files. |  |
| [OpenBazaar](https://openbazaar.org/) | [Community]() | Create a store. Sell whatever you’d like. Reach a new audience. Get paid in cryptocurrency |  |
| [PassBolt](https://www.passbolt.com/) | []() |  |  |
| [PhotoPrism](https://github.com/photoprism/photoprism) | []() |  |  |
| []() | []() |  |  |
| []() | []() |  |  |



| []() | []() |  |  |

- Log viewer web app /[splunk](http://www.splunk.com/en_us/products/splunk-enterprise/free-vs-enterprise.html)/fluentd webui/ loggly aggregator/Graylog/rtail
- SubSonic/[Madsonic](https://github.com/mediadepot/docker-madsonic)/koel for Music/Mopidy/sonerezh/cloudtunes
- Huginn/Bip Automation
- SqlPad
- visallo
- dillinger
- alltube
- bookstack
- tagspaces
- dashboard/[muximux](https://github.com/mescon/Muximux)
- monica - personal relationship manager
- [lidarr](https://github.com/lidarr/Lidarr)
- Ideas for other containers - https://github.com/DFabric/DPlatform-ShellCore https://github.com/Kickball/awesome-selfhosted#read-it-later-lists


In addition to the software you see mentioned above, you should take a look at the full catalog of Docker Images maintained by [LinuxServer.io](https://www.linuxserver.io/).
They are a community run organization that maintains FOSS Docker Images for the community.


# Installation

## CoreOS Ignition

# Screenshots

Click to view the entire album.

[![](https://i.imgur.com/cl5ReOY.jpg)](https://imgur.com/a/VMcMtVY)


# FAQ
## Maintenance
- Assumes that all drives have been formatted and mounted
- how to handle mounting drives. Should they be mounted via chef. If  yes, how do we configure all the various options for each drive, if no, how do we select all the mounted drives to use with greyhole.


## Backups


# Issues

# Future Ideas

# References






---

> ALL DOCUMENTATION BELOW HERE IS OLD AND NEEDS TO BE RE-WRITTEN AND MOVED ABOVE.



- MOTD: https://github.com/yboetz/motd




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

# Logging
- http://blog.treasuredata.com/blog/2015/08/03/5-use-cases-docker-fluentd/

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

