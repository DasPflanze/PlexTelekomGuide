# Plex + TVHeadend & TVHProxy Setup in Docker

In the following parts i'm going to explain how to setup Plex with TVHeadend using TVHProxy in docker. Additional I'll show you how to use these containers to receive Entertain IPTV with Plex DVR.

**Requirements:**

* Telekom Entertain IPTV access
* Installed Docker Engine

### My own setup

I'm using

* a [iBASE MI980](https://www.ibase.com.tw/english/ProductDetail/EmbeddedComputing/MI980) (i7-4700EQ 8x2,40Ghz)
* equipped with 2x 8 GB of [Crucial 1600 DDR3L SO DIMM](http://www.crucial.de/deu/de/ct102464bf160b)
* and a Digital Devices Cine CT
* with 3x WD Red 3TB Drives
* in a [Intertech SC4100 case](https://www.inter-tech.de/products/ipc/storage-cases/sc-4100)
* powered by a [Intertech AP-MFATX25P8 Flex ATX PSU](https://www.inter-tech.de/products/psu/server-psu/ap-mfatx25p8).

On this setup I've installed a Proxmox instance to virtualize a [Freenas](http://www.freenas.org) KVM and a Ubuntu KVM with installed Docker. Also I am now able to setup diffrent types of servers (Minecraft, TeamSpeak) and services (OwnCloud, NextCloud, MediathekView) without the need of renting a Virtual Private Server.

As addition to the following setup guide I would like to share my filesystem with you.
My 3x WD Drives (RaidZ1) are shared through freenas via NFS to my Ubuntu KVM so my docker containers can access my `Multimedia`-Dataset and my `Docker`-Dataset. In my Multimedia Share are all my Movies and Series saved, whereas my Docker Share stores all of my container-related data.

## 1. Setting up Plex in Docker

### Creation 
**Used Container:** [linuxserver/plex](https://github.com/linuxserver/docker-plex)

To create the Plex container change following command to your needs. For more information read the readme on [github](https://github.com/linuxserver/docker-plex). 

* `--net=host` **Required** - don't change.
* `-e TZ=Europe/Berlin` your timezone ([Full list](https://www.vmware.com/support/developer/vc-sdk/visdk400pubs/ReferenceGuide/timezone.html))
* `-e VERSION=latest` options are *latest, public,* specific version (eg *0.9.12.4.1192-9a47d21*) [(readme)](https://github.com/linuxserver/docker-plex#setting-up-the-application)
* `-e PUID=1001 -e PGID=1001` at best not needed, at worst needed ([readme](https://github.com/linuxserver/docker-plex#user--group-identifiers))
*  `-v </path/to/library>:/config` change path (left) on host to your needs - can grow very large according to library size (if available, place on SSD for better overall performance)
*  `-v <path/to/tvseries>:/data/tvshows ` change path (left) on host to your needs (use multiple times for more folders or map main folder to `:/data`)
*  `-v </path for transcoding>:/transcode` change path (left) on host to your needs (if available, place on SSD for better transcoding performance)


```
docker create \
   --name=plex \
   --net=host \
   -e VERSION=latest \
   -e PUID=<UID> -e PGID=<GID> \
   -e TZ=<timezone> \
   -v </path/to/library>:/config \
   -v </path/to/tvseries>:/data/tvshows \
   -v </path/to/movies>:/data/movies \
   -v </path for transcoding>:/transcode \
   linuxserver/plex
```

>### My Command
>
>```
>docker create \
   --name=plex \
   --net=host \
   -e VERSION=latest \
   -e PUID=0 -e PGID=1002 \
   -e TZ=Europe/Berlin \
   -v /mnt/freenas/Docker/plex/config:/config \
   -v /mnt/freenas/Multimedia/:/data \
   linuxserver/plex
>```

### Starting
Start the new created container.

```
docker start plex
```
Every time the container starts it will search for plex updates and install the newest version. If you own PlexPass it will install the latest PlexPass update after first login and a following restart. (if VERSION is set accordingly.)

After startup is done, you can access the **WebUI** at `<ip-address>:32400`

To get the IP address of the container use 

```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' plex
```
### Setup

Now setup your Plex instance after your requirements. Your files are located at the with `-v` specified paths (`<path/on/host>:<path/in/container>`)


## 2. Setting up TVHeadend in Docker

###Creating
**Used container:** [linuxserver/tvheadend](https://github.com/linuxserver/docker-tvheadend)

To create the TVHeadend Container you need to edit following command to suit your needs.

* `--net=host` if you won't use IPTV, SAT>IP or HDHomeRun `--net=bridge` is also possible.
* `-v </path/to/data>:/config` change path (left) on host to your needs
* `-v </path/to/recordings>:/recordings` change path (left) on host to your needs - **not** required if Plex DVR in use
* `-e PUID=1001 -e PGID=1001` at best not needed, at worst needed ([readme](https://github.com/linuxserver/docker-tvheadend#user--group-identifiers))
* `-e RUN_OPTS=<parameter>` not needed ([more info](https://github.com/linuxserver/docker-tvheadend#additional-runtime-parameters))
* `-p 9981:9981` & `-p 9982:9982` only needed if `--net=bridge` is set
* `--device=/dev/dvb` to passtrough DVB cards connected to the host

```
docker create \
  --name=tvheadend \
  --net=host \
  -v </path/to/data>:/config \
  -v </path/to/recordings>:/recordings \
  -e PGID=<gid> -e PUID=<uid>  \
  -e RUN_OPTS=<parameter> \
  -e TZ=<timezone> \
  --device=/dev/dvb
  linuxserver/tvheadend
```

>### My Command
>```
>docker create \
  --name=tvheadend \
  --net=host \
  -v /mnt/FreeNAS/Docker/tvheadend/config:/config \
  -v /mnt/FreeNAS/Docker/tvheadend/recordings:/recordings \
  -e PGID=1002 -e PUID=0  \
  -e TZ=Europe/Berlin \
  -p 9981:9981 \
  -p 9982:9982 \
  linuxserver/tvheadend
>```

###Starting

Start the new created container

```
docker start tvheadend
```

###Setup

At first start you will see a **wizard** when accessing the WebUI at `<ip-address>:9981`.

To get the IP address of the container use 

```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' plex
```

Your wizard will look something like this:

![TVH Wizard 1](assets/img/tvh-wizard-1.png)
Configure your web and EPG Language as needed.


![TVH Wizard 2](assets/img/tvh-wizard-2.png)
Set admin login (your later authentification to the WebUI) and user login (to give TVHProxy access to streams).

![TVH Wizard 2](assets/img/tvh-wizard-3.png)
Don't mind the additional DVB-C and DVB-T Tuners of my DigitalDevices Tuner Card. If you're only going to use Telekom Entertain you won't see any of them and you just need to set the `Network type` of `Tuner: IPTV` to `IPTV Automatic Network`
