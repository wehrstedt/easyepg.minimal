![GitHub release](https://img.shields.io/github/release/dlueth/easyepg.minimal.svg)
![Docker Pulls](https://img.shields.io/docker/pulls/qoopido/easyepg.minimal.svg)

# easyepg.minimal
A minimal docker container for running easyepg either on demand or permanently with built-in cronjob

## Prerequisites
You will need to have `docker` installed on your system and the user you want to run it needs to be in the `docker` group.

## Installation
As root user issue the following commands line by line to download easyepg.minimal's utility bash script, make it globally available and pull the image from the docker repository
```
curl -s https://raw.githubusercontent.com/dlueth/easyepg.minimal/master/easyepg.minimal > /usr/local/sbin/easyepg.minimal
chmod +x /usr/local/sbin/easyepg.minimal
docker pull qoopido/easyepg.minimal:latest
```

The image is a multi-arch build providing variants for amd64, arm32v7 and arm64v8 - the correct variant for your architecture should<sup>TM</sup> be pulled automatically.

## Initial setup
Switch to the user you want to run the container with, create a directory to permanently store easyepg, create containers, start the admin container and enter it via
```
mkdir ~/easyepg
easyepg.minimal -m create -v ~/easyepg
docker start easyepg.admin
docker exec -ti easyepg.admin /bin/bash
```

After you successfully switched into the container issue
```
su - easyepg
./epg-sh
```

to start easyepg's setup and configure it. When your setup is finished return to the shell and issue `exit` to leave the container followed by `docker stop easyepg.admin` to stop it. 

## Updating EPG XML-files

### Variant A: via Cronjob in the container
Simply run the following command while logged in as the desired user

```
docker start easyepg.cron
```

There already is a crontab in the container that will run easyepg at 2:00am every night.

### Variant B: via Cronjob on the host
> Skip this section if you decided to go with Variant A (e.g. you are running the container on a NAS)

```
crontab -e
```

Append the following lines to the file that should have been opened and replace `[your file]` with the filename of your generated XML

```
0 2 * * * docker start easyepg.run
0 4 * * * cat ~/easyepg/xml/[your file].xml | socat - UNIX-CONNECT:/home/hts/.hts/tvheadend/epggrab/xmltv.sock
10 4 * * * cat ~/easyepg/xml/[your file].xml | socat - UNIX-CONNECT:/home/hts/.hts/tvheadend/epggrab/xmltv.sock 
```

Save and exit the file and you are done!

## Limiting CPU usage
During the initial setup (see above) when you run

```
easyepg.minimal -m create -v ~/easyepg 
```

CPU usage of the created containers will be limited to 0.5 * number of cores by default. So on a machine with 4 cores the container will be limited to 2 cores.

If you want to utilize more or less CPU cores you may add a `-r` to any `easyepg.minimal -m create -v ~/easyepg` call and set its value to a positive float with a maximum of `1`. You can re-create easyepg.minimal's containers at any time if none of them is running.

So, e.g.

```
easyepg.minimal -m create -v ~/easyepg -r 0.75
```

will allow the container to use 3 cores on a machine with 4 cores.
