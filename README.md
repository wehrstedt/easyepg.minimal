# easyepg.minimal
A minimal docker container for running easyepg

## Prerequisites
You will need to have `docker` installed on your system and the user you want to run docker under needs to be in the `docker` group.

## Installation
As root user issue the following commands line by line to download a script and make it globally available:

``` 
curl -s https://raw.githubusercontent.com/dlueth/easyepg.minimal/feature/nas-support/eemd > /usr/local/sbin/eemd
chmod +x /usr/local/sbin/eemd
```

## Setup & Administration
Switch over to the user you want to run docker under and create (e.g.) a directory `easyepg` in its home folder.

Afterwards run `eemd admin -v ~/easyepg` to enter the docker container in admin-mode. When you finally see the container's prompt (it will download the image from docker on first run) issue `su - easyepg` followed by `./epg.sh` to start easyepg's setup.

When you are finished setting easyepg up to your liking exit the running docker container by issuing `exit`.

## First run
There are two ways of running the container depending on your surrounding environment/host.

### Directly 
Issue `eemd run -v ~/easyepg` from your command prompt to manually test if everything is working as expected. If it does you might want to create a cronjob on your local host machine:

Still as the user you would like to run docker under issue `crontab -e` and put in the following lines

```
0 2 * * * /usr/local/sbin/eemd run -v ~/easyepg
0 4 * * * cat ~/easyepg/xml/[your file].xml | socat - UNIX-CONNECT:/home/hts/.hts/tvheadend/epggrab/xmltv.sock
10 4 * * * cat ~/easyepg/xml/[your file].xml | socat - UNIX-CONNECT:/home/hts/.hts/tvheadend/epggrab/xmltv.sock
```

### NAS-System
On most NAS systems supporting docker, containers are supposed to be "always running" so the direct approach will most likely not work here.

There is another mode built-in to support this type of system:

Issue `eemd cron -v ~/easyepg` to start the container updating epg information at 2am in the morning. This will take care of most things automatically.

If you are unable to run a container via shell script you may as well run it directly via

```
docker run --rm -ti -d \
  -e "MODE=cron" \
  -e "TZ=${TZ}" \ # Timezone, defaults to Europe/Berlin
  -e "PGID=${PGID}" \ # Group-ID, defaults to 1099 
  -e "PUID=${PUID}" \ # User-ID, defaults to 1099
  -v ${VOLUME}:/easyepg \ # Absolute (!) path to a shared directory storing easyepg & its settings
  --name easyepg-cron qoopido/easyepg.minimal:1.0.6-rc.3
```

### Limiting CPU usage
When using the `eemd` CPU usage of the started containers will be limited to 0.5 * number of cores by default. So on a machine with 4 cores the container will be limited to 2 cores by default. If you want to utilize more or less CPU cores you may add a `-r` to any `eemd` call and set its value to a positive float with a maximum of `1`.  
