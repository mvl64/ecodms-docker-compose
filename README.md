# ecodms-docker-compose
Running Ecodms in a docker environment, using docker compose, on a Synology NAS.

This docker-compose file creates a container for ecodms 21.12 with persistent data store.
In my setup, data for each new version is stored in a separate data store - so I can easily test the new version and revert back to the old container if needed.

You can either take the storage layout, or choose your own folders by editing the docker-compose file.

## Data storage
Ecodms document and database data will be stored in a subfolder called ```/docker/ecoDMS21.12```

Next to the container data - I have one generic folder related to ecoDMS.
This folder contains a subfolder for the scaninput: ```/ecoDMS/ScanInput```.
It also contains subfolders for backups and restores (see next section)

## Backup and Restore
Ecodms has the capability to easily create backups which can easily be restored. For this there exist two folders, which in this case are placed under ```/ecoDMS/Backup``` and ```/ecoDMS/Restore```

If you want to backup your ecodms container data, just create an empty file called "create" in the backup folder, shortly after the backup will be created as a zip file and then the "create" file will be removed

For example: ```touch /ecoDMS/Backup/create```

If you want to restore a backup then place one of the *.zip files that you have created in the folder ```/ecoDMS/Restore``` and rename it to ```restore.zip```.
Then restart the container with ```docker-compose down && docker-compose up -d``` and the restore will be performed

**BUT BE AWARE,  this will overwrite *ALL DATA* in the ecodms container with the data from ```restore.zip```**

## Ports
Appliaction ports can also be adjusted to your likings by editing the ```ports:``` part in the docker-compose file
- Port 17001 which is mapped to 17001 is the connection port for the ecodms Client software
- Port 8080 which is mapped to 17004 is the web application connection port for the ecodms Web Client (this must be activated first in the application to be used
- Port 8180 which is mapped to 17005 is the ecodms API connection port for API calls

*Note: By default connections are not encrypted.*

If you want an encrypted connection to your cloud ecodms, you have to use some kind of VPN tunnel, for example Softether, OpenVPN, IPsec, etc., you could also use an SSL encrypted reverse proxy, as mentioned below to make a secure HTTPS connection to the ecodms Web Client

## Webclient / Reverse Proxy
If you would like to run ecodms Webservice on your own server, e.g. in the cloud, I have attached a nginx reverse proxy configuration file, which you could edit to your likings and then place under ```/etc/nginx/sites-available```.
Then you can link it to ```/etc/nginx/sites-enabled``` whenever you want to activate it.

For example with this command: ```ln -s /etc/nginx/sites-available/ecodms-reverse-proxy.conf /etc/nginx/sites-enabled/ecodms```
Nginx configuration can be tested via ```nginx -t```

To make this work, you have to make sure that you have your own webserver certificates in place, maybe Let's Encrypt and you have to create the DH paramater file for nginx if it not already exists, instructions are in the comments of the file ```ecodms-reverse-proxy.conf```
