# Plex over Docker on Raspberry

Build your own [plex](https://www.plex.tv/) server with [transmission](https://transmissionbt.com/) and [flexget](https://flexget.com/) on your Raspberry from scratch.

Transmission media files downloads are automatically published on plex using flexget.

Totally based on peladonerd [video](https://www.youtube.com/watch?v=TqVoHWjz_tI) and his forked repo.

Tested on Raspberry Pi 4.

## Requirements

You need a running Raspberry Pi to install the docker containers.

Here we install the Raspbian 64bits OS. Steps:
 
1. Download latest Raspbian 64bits image from [here](https://downloads.raspberrypi.org/raspios_arm64/images/).
2. Write the image to an SD card. Recommended: use [Raspberry Pi Imager](https://www.raspberrypi.org/software/) with `Use Custom` option.
3. Enable ssh by default, adding an file ``ssh`` empty file to the SD card on `bootfs` partition.
4. Connect the Pi with cable and turn on.
5. Discover IP from your router and access via ssh. Default user ``pi`` with password ``raspberry``.
6. Change ``pi`` user password.
```
passwd
```
7. Update the system.
```
apt update
apt upgrade
```
Then you can mount an USB disk (or anything else) where store all your stuff.

## Install

To install plex, transmission and flexget run this steps:
1. Clone this repo.
```
git clone https://github.com/jmformenti/plex-rpi.git
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
	1. Sets transmission password, `rpc-password` parameter in `transmission/settings.json` file.
	2. Sets transmission password in flexget, `transmission.pwd` parameter in `flexget/variables.yml` file.
4. Configure flexget in `flexget/config.yml` as you need. Recommended:
	1. Add your series in `templates.tv.series.tv`.
	2. Add your torrent feeds in `tasks`.

## Executing

Once installed, just run this command to start all containers:
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
http://host:32400 
where `host` is your `NETWORK_PLEX_IP` parameter.

First time you access you'll have to complete the configuration with the setup wizard.

### Transmission
http://host:9091/transmission/web 
where `host` is your Pi ip.

### Flexget
http://host:5050 
where `host` is your Pi ip.

## Troubleshooting

### Flexget is not running
Check logs to see the problem.
```
docker logs plex-rpi_flextget_1
```
If you see this error "" try to put a more complex password.

### Avoid online transcode video

By default, plex try to transcode video to save network bandwith but this can put our Pi in troubles. To avoid this you have to configure **all the clients** (web, android, ...) to use max resolution. 

For example in plex web:
1. Go to `Settings`.
2. Go to `Quality` section.
    1. Turn off `Automatically Adjust Quality` check.
    2. Put `Internet Streaming` to max value.
    3. Put `Home Streaming` to max value.
