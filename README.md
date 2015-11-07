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
- Guacamole/VNC/Teamviewer
- Samba (to mount the Greyhole shares locally)
- Dynamic DNS updater script/DuckDNS
- Chefdk
- Docker
- SMART disk status

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
- Log viewer web app /fluentd webui/ loggly aggregator/Graylog
- BTSync/SyncThing
- Backup service
- Update checker service
- PopcornTime

# Container Configuration
- All containers will have configuration service to dynamically generate config files:
  - https://github.com/jwilder/docker-gen
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
- Hosts file entry


# References
- http://www.cyberciti.biz/tips/spice-up-your-unix-linux-shell-scripts.html
- https://www.reddit.com/r/programming/comments/1pnkxs/dont_pipe_to_your_shell/cd4bu30
- https://github.com/jwilder/docker-gen
- https://github.com/hashicorp/consul-template
- https://github.com/markround/tiller
- https://github.com/gliderlabs/registrator
- https://en.wikibooks.org/wiki/Bash_Shell_Scripting/Whiptail
- http://tldp.org/LDP/abs/html/here-docs.html



