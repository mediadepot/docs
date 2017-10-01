# Planning
This is a organization to group all the cookbooks and docker files required to setup up a self-hosted media server with the following capabilities:
- Some form of JBOD disk storage (most likely greyhole as that's what I'm currently using)
- Media server applications such as plex, sickbeard, couchpotato, etc to manage and view media
- Utility applications such as ajenti, openvpn, conky, btsync, bittorrent, vnc. 
- Notifications system (so that you are notified whenever any service stops or starts, and when media is added)

This is a docker version of the SparkTree.Chef.Nas cookbook. 

# Assumptions
Here are a couple of assumptions we are making:
- The server will be self hosted, with only one node (if you need a multihosted media server, this wont work for you)
- Drive storage will be configured using a JBOD array that will be mounted on the host, and shared with the docker containers
- A Github gist will be used to persistently store secrets and backup data
- Pushover will be used to send remote notifications
- Assumes that all drives have been formatted and mounted 

# Host Applications
The following software will run on the host:
- JBOD/Union FS (Greyhole/Mergerfs)
- SSH daemon
- OpenVPN
- VNC server
- Samba
- Dynamic DNS updater script/DuckDNS
- Chefdk
- Docker
- SMART disk status

# Docker Containers
The following software will run in docker containers:
- Vault or other credential/secret management system
- Rancher
- Ajenti? (does this make sense, or should it just be run on the Host)
- [Couchpotato](https://github.com/mediadepot/docker-couchpotato)
- [Deluge](https://github.com/mediadepot/docker-deluge)/Hadouken/ruTorrent
- [Headphones](https://github.com/mediadepot/docker-headphones)
- [Plex](https://github.com/mediadepot/docker-plex)/Emby
- [Sickrage](https://github.com/mediadepot/docker-sickrage)/Sickbeard
- [Headphones](https://github.com/mediadepot/docker-headphones)
- Nginx/HAProxy router
- Log viewer web app /[splunk](http://www.splunk.com/en_us/products/splunk-enterprise/free-vs-enterprise.html)/fluentd webui/ loggly aggregator/Graylog
- [BTSync](https://github.com/mediadepot/docker-btsync)/SyncThing/Seafile/Pydio/Nextcloud
- Backup service
- Update checker service
- Lychee/Photato/Koken.me
- [Guacamole (VNC)](https://github.com/mediadepot/docker-guacamole)
- [LazyLibrarian](https://github.com/mediadepot/docker-lazylibrarian)
- OpenVPN_AS
- StremIO
- Status Page/Dashboard - Dashing/Atlasboard
- Sandstorm
- openbazaar
- SubSonic/[Madsonic](https://github.com/mediadepot/docker-madsonic)/Sonarr/koel for Music/Mopidy/sonerezh/cloudtunes
- Huginn/Bip Automation
- SqlPad
- visallo
- dillinger
- [jDownloader](https://hub.docker.com/r/aptalca/docker-jdownloader2/)
- alltube
- Ideas for other containers - https://github.com/DFabric/DPlatform-ShellCore

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

