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

# Host Applications
The following software will run on the host:
- Greyhole (which will aggregate all drive storage)
- SSH daemon
- OpenVPN
- VNC
- Samba (to mount the Greyhole shares locally)
- Dynamic DNS updater script
- Chefdk
- Docker

# Docker Containers
The following software will run in docker containers:
- Vault or other credential/secret management system
- Rancher
- Ajenti? (does this make sense, or should it just be run on the Host)
- Couchpotato
- Deluge/Transmission
- Headphones
- Plex
- Sickrage/Sickbeard
- Headphones
- Nginx/HAProxy router
- Log viewer web app /fluentd webui
- BTSynx
- Backup service
- Update checker service

# Installation

