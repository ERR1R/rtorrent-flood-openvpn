# rTorrent + FloodUI + OpenVPN
Docker container for rTorrent + FloodUI with integrated OpenVPN client.

It is based on the Rockylinux 8.5 docker image:

## What does this image?
The container connects at startup during the boot process to the OpenVPN service of your choice. After the OpenVPN client connected successfully, the rTorrent and FloodUI service will startup.

![FloodUI](https://github.com/h1f0x/rtorrent-flood-openvpn/blob/master/images/1.png?raw=true) 

## Install instructions

### Important
Go to http://host:8000/flood and use

User: Torrent
Password: Torrent
Connect: 127.0.0.1:5000

> In case it doesn't auto connect....

### Docker volumes
The following volumes will get mounted:

- /path/to/config:/config
- /path/to/output/incomplete:/output/incomplete
- /path/to/output/complete:/output/complete


### OpenVPN configuration
Prepare an OpenVPN configuration of your choice. An automated login by username/password is also possible with the "user-pass-auth" parameter in the client.conf

> Should no configuration be present at the first run, an example config will be deployed at the mounted /config folder which can be edited.

The OpenVPN service will be verified every 60s. If it's not running anymore it will restart the connection.

### Deploy the docker container
To get the docker up and running execute fhe following command:

```
sudo docker run -it --privileged --name rtorrent-flood-openvpn -v /path/to/config:/config -v /path/to/output:/output -d -p 8000:80 -p 8080:8080 err1r/rtorrent-flood-openvpn
```
> If not done already, deploy or modify the OpenVPN client.conf at /path/to/config/vpn

```
After your changes restart the container.

docker restart rtorrent-flood-openvpn
```

### Verify OpenVPN status
In "/config/my-external-ip.txt"  the current external ip address can be found. The file will be updated every 60s.

### Sonarr Support
You can use Sonarr with this client as well. Configure your Sonarr with the following params:

```
# Normal Container
Name: rflood-openvpn
Enable: Yes
Host: <IP> or <HOSTNAME>
Port: 8080
Username & Password: empty

# PGBlitz
Name: rflood-openvpn
Enable: Yes
Host: rflood-openvpn
Port: 8080
Username & Password: empty
```

### Tagging
This docker container supports tagging when feeding new torrents.

If a tag is set, the torrent will be copied to the following location once it's finished:

```
/output/complete/{tag}
```

If no tag is set, the default location is:

```
/output/complete/unsorted
```

## Configuration files

Several configuration files will be deployed to the mounted /config folder:

| Folder | Description |
| :--- | :--- |
| flood/* | flood default db / user file |
| rtorrent/* | rtorrent.rc, session data, *.torrent files, etc. |
| vpn/* | vpn default config / user config |

### FloodUI default settings
> The default login for FloodUI is `Torrent` : `Torrent`

Please change the username : password in the settings.

The configured socket is `scgi_port = 0.0.0.0:5000`

### rTorrent default settings

#### Listening port for incoming peer traffic
```
network.port_range.set = 50000-50000
network.port_random.set = no
```
#### Check the hash after the end of the download
```
check_hash = no
```
#### Peer settings
```
throttle.max_uploads.set = 100
throttle.max_uploads.global.set = 250
throttle.min_peers.normal.set = 20
throttle.max_peers.normal.set = 60
throttle.min_peers.seed.set = 30
throttle.max_peers.seed.set = 80
trackers.numwant.set = 80
```
#### Encryption
```
protocol.encryption.set = allow_incoming,try_outgoing,enable_retry
```
#### Limits for file handle resources
```
network.http.max_open.set = 50
network.max_open_files.set = 600
network.max_open_sockets.set = 300
```
#### Memory resource usage
```
pieces.memory.max.set = 1800M
network.xmlrpc.size_limit.set = 12M
```

#### Basic operational settings 
```
session.path.set = (cat, (cfg.session))
directory.default.set = (cat, (cfg.download))
log.execute = (cat, (cfg.logs), "execute.log")
log.xmlrpc = (cat, (cfg.logs), "xmlrpc.log")
execute.nothrow = sh, -c, (cat, "echo >",\
    (session.path), "rtorrent.pid", " ",(system.pid))
```
#### Other operational settings
```
encoding.add = utf8
system.umask.set = 0027
system.cwd.set = (directory.default)
network.http.dns_cache_timeout.set = 25
schedule2 = monitor_diskspace, 15, 60, ((close_low_diskspace, 1000M))
method.insert = system.startup_time, value|const, (system.time)
method.insert = d.data_path, simple,\
    "if=(d.is_multi_file),\
        (cat, (d.directory), /),\
        (cat, (d.directory), /, (d.name))"
method.insert = d.session_file, simple, "cat=(session.path), (d.hash), .torrent"
```
#### Watch directories
```
## Add torrent
schedule2 = watch_load, 11, 10, ((load.verbose, (cat, (cfg.watch), "load/*.torrent")))
## Add & download straight away
schedule2 = watch_start, 10, 10, ((load.start_verbose, (cat, (cfg.watch), "start/*.torrent")))
```
#### Move on finished
```
method.insert = d.get_finished_dir,simple,\
        "if=(d.get_custom1),\
        (cat, /output/complete/, (d.get_custom1), /),\
        (cat, /output/complete/unsorted/)"
method.set_key = event.download.finished,move_complete,"d.stop=;execute=mkdir,-p,$d.get_finished_dir=;execute=cp,-fr,$d.get_base_path=,$d.get_finished_dir=;d.start=;d.hash"
```
#### Socket specs
```
scgi_port = 0.0.0.0:5000
```
#### Ratio trigger
```
method.set = group.seeding.ratio.command, "d.close="
```
#### Logging
```
print = (cat, "Logging to ", (cfg.logfile))
log.open_file = "log", (cfg.logfile)
log.add_output = "info", "log"
#log.add_output = "tracker_debug", "log"
```
#### Rflood not working
```
Check if you have enabled the Privileged mode for this container
```

## Enjoy!

Open the browser and go to:

> http://localhost:8000/flood
