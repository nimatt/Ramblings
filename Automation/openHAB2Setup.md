# openHAB 2 setup on a Raspberry pi 3
I don't think anyone has missed the increasingly popular trend of home automation.
Personally I've been scared of by the multitude of security flaws found in these devices.
But then I found openHAB, a system meant to be run on your own hardware without any
dependency on any cloud service

## What is openHAB?
OpenHAB is a software that enables you to gather information and relay commands
to a multitude of different units. It does not have any hardware but instead support
communications with devices from many different vendors such as RFXCom and Sonos.

It is however not only a but you can also
* Configure several storages such as rrd4j, MapDB and MySQL
* Write rules that triggers on time, system or item events

A lot more information can be found at [docs.openhab.org](http://docs.openhab.org/)

## Installation on a Raspberry
### Basic install
Basic instructions for installation in the Raspberry can be found in the [documentation](http://docs.openhab.org/installation/rasppi.html) so I will not go into details
(if I get bored some day I might come back and elaborate)

1. Prepare the micro-SD card with NOOBS or Raspbian
1. Boot the Raspberry and perform the Raspbian installation
1. Open a terminal in Raspbian
1. Change some system settings ```sudo raspi-config```
 * Expand the file system
 * Change the password
 * Change the hostname (if you'd like)
 * Change the memory split for the GPU to 16
1. Restart the pi
1. Open a terminal
1. Set the correct time zone

 ```
 dpkg-reconfigure tzdata
 ```

1. Update Raspbian

 ```
 sudo apt-get update
 sudo apt-get upgrade
 ```

1. Install some basic tools (you can skip this if youäd like)

 ```
 sudo apt-get install screen mc emacs git htop
 ```

1. Check your java version

 ```
 java -version
 ```

1. If it is lower than 1.8 then install it

 ```
 sudo apt-get install oracle-java8-jdk
 ```

1. Add openHAB 2 repository to apt. There is at this time (2016-10-16) no more
than beta releases of openHAB 2 so
I decided to use the snapshot release which is being build on every update. If a
more stable release has been made the procedure is likely the same but the urls in
the following commands has changed.

 ```
 echo 'deb https://openhab.ci.cloudbees.com/job/openHAB-Distribution/ws/distributions/openhab-offline/target/apt-repo/ /' | sudo tee /etc/apt/sources.list.d/openhab2.list
 echo 'deb https://openhab.ci.cloudbees.com/job/openHAB-Distribution/ws/distributions/openhab-online/target/apt-repo/ /' | sudo tee --append /etc/apt/sources.list.d/openhab2.list
 ```

1. Add the public key to apt

 ```
 wget -qO - 'http://www.openhab.org/keys/public-key-snapshots.asc' | sudo apt-key add -
 ```

1. Add support for https in apt

 ```
 sudo apt-get install apt-transport-https
 ```

1. Fetch the new information

 ```
 sudo apt-get update
 ```

1. I had some issues with getting the _online_ version to work so I took the easy route and installed the offline version

 ```
 sudo apt-get install openhab2-offline
 ```

1. Start the system

 ```
 sudo systemctl start openhab2.service
 ```

1. Check the status using

 ```
 sudo systemctl status openhab2.service
 ```

1. Configure it to start automatically on Respberry startup

 ```
 sudo systemctl daemon-reload
 sudo systemctl enable openhab2.service
 ```

1. Wait... (first startup takes a while and you'll only get a 404 for checking too soon)
1. Point your browser to http://your_raspberrys_ip_address:8080 (if your on your Raspberry that'll be http://127.0.0.1:8080)
 * If you don't know the ip of the raspberry you can get it using ```ifconfig```

## Quick info about nomenclature (things, items, ...)
...

## Keep track of your config
...

## Configure Sonos
...

## Configure RFXCom transeiver
...

## Monitoring temperature and humidity
...

## Configure NEXA lighting
...

## Using it as an alarm clock
...
