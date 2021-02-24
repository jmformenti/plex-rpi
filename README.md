# Plex over Docker on Raspberry

Build your own [plex](https://www.plex.tv/) server with [transmission](https://transmissionbt.com/) and [flexget](https://flexget.com/) on your Raspberry from scratch.

Transmission media files downloads are automatically published on plex using flexget.

Totally based on peladonerd [video](https://www.youtube.com/watch?v=TqVoHWjz_tI) and his [repo](https://github.com/pablokbs/plex-rpi).

Tested on Raspberry Pi 4.

## Requirements

Before starting you need a 64bits OS with docker installed on your Raspberry Pi.

If not, you can follow this few steps to install and configure the Raspbian 64bits OS on your Pi:
 
1. Download latest Raspbian 64bits image from [here](https://downloads.raspberrypi.org/raspios_arm64/images/).
2. Write the image to an SD card. Recommended: use [Raspberry Pi Imager](https://www.raspberrypi.org/software/) with `Use Custom` option.
3. Enable ssh by default, adding an empty file named ``ssh`` to the SD card on `boot` partition.
4. Connect the Pi with cable and turn on.
5. Discover IP from your router and access via ssh. Default user ``pi`` with password ``raspberry``.
6. Change ``pi`` user password.
```
passwd
```
7. Configure timezone (for example, Europe/Andorra).
```
timedatectl set-timezone Europe/Andorra
```
8. Configure static IP (for example, asigning 192.168.1.2 as ip in 192.168.1.* network):
```
interface eth0
static ip_address=192.168.1.2/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```
9. Reboot and access via ssh using the new IP.
```
reboot
```
10. Update the system.
```
sudo apt update
sudo apt upgrade
```
11. Install ``docker`` and ``docker-compose``.
```
sudo apt install docker docker-compose
```
12. Add ``pi`` user to docker group.
```
sudo usermod -aG docker pi
```
13. Enable and start docker service.
```
sudo systemctl enable --now docker.service
```

Then you can [mount an USB disk](https://www.raspberrypi.org/documentation/configuration/external-storage.md) (or anything else) where store all your stuff.

**Tip**: To extend the life of your SD it is recommended to use a directory of your external storage for the temporary docker directory. Add this line to ``/etc/defaul/docker`` file using your dir:
```
export DOCKER_TMPDIR=/path/to/docker/tmp/dir
```

## Install

To install plex, transmission and flexget run this steps:
1. Clone this repo and enter in ``plex-rpi`` directory.
```
git clone https://github.com/jmformenti/plex-rpi.git
cd plex-rpi
```
2. Configure main parameters inside `.env` file.
	* **UID**. User ID who runs the dockers (typically `pi` user, run `id -u pi`).
	* **GID**. Group ID who runs the dockers (typically `pi` group, run `id -g pi`).
	* **MEDIA**. Root directory where films, series, etc are saved.
	* **STORAGE**. Root directory for torrents and temporal files.
	* **NETWORK_SUBNET**. Configure your subnet (for example, 192.168.1.0/24).
	* **NETWORK_GATEWAY**. Configure your gateway (for example, 192.168.1.1).
	* **NETWORK_PLEX_IP**. Configure the IP to user for plex server (for example, 192.168.1.3).
	* **FLEXGET_PWD**. Password for flexget.
3. Configure transmission password.
	* Sets transmission password, `rpc-password` parameter in `transmission/settings.json` file.
	* Sets transmission password in flexget, `transmission.pwd` parameter in `flexget/variables.yml` file.
4. Configure flexget in `flexget/config.yml` as you need. Recommended:
	* Add your series in `templates.tv.series.tv`.
	* Add your torrent feeds in `tasks`.

## Executing

Once installed, just run this command (inside ``plex-rpi`` dir) to start all containers:
```
docker-compose up -d
```
To stop all containers run:
```
docker-compose stop
```
If you want remove all containers run:
```
docker-compose down
```

## Accessing

### Plex
http://host:32400/web/index.html 
where `host` is your `NETWORK_PLEX_IP` parameter.

First time you access you'll have to complete the configuration with the setup wizard.

### Transmission
http://host:9091/transmission/web/ 
where `host` is your Pi ip.

### Flexget
http://host:5050 
where `host` is your Pi ip.

## Troubleshooting

### Flexget is not accessible
Check logs to see the problem.
```
docker logs plex-rpi_flexget_1
```
If you see this error "Password 'XXXXX' is not strong enough. Suggestions: Add another word or two. Uncommon words are better." try to put a more complex password.

### Avoid online transcode video

By default, plex try to transcode video to save network bandwith but this can put our Pi in troubles. To avoid this you have to configure **all the clients** (web, android, ...) to use max resolution. 

For example in plex web:
1. Go to `Settings`.
2. Go to `Quality` section.
    * Turn off `Automatically Adjust Quality` check.
    * Put `Internet Streaming` to max value.
    * Put `Home Streaming` to max value.
