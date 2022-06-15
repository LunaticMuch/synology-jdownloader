# JDownloader configuration for DSM7

This document helps with the setup of a startup script for running (JDownloader)[https://jdownloader.org] on Synology with DSM7

## Intro

DSM7 uses `systemd` for services statup operations. Systemd allows additional scripts and complex chaining of these. While systemd can be irksume manytimes, it's quite powerful and it is better to use it. This ensures any future update of DSM7 won't break your script and the startup.

## Prerequisites

This page will help you to configure your Synology to automatically start, restart, stop and reload your JDownloader. Although, it requires you the following:

- You need admin rights on your Synology
- Your Synology must be connected to internet
- You need a basic understand of how a terminal works
- You need already to have JDownloader installed on your Synology
- You need to be able to connect from your computer to the Synology using SSH (if on Windows, check (Putty)[https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html])

## Usage

### Setting up Java JRE/JDK locally

_You can skip this section if you already have Java installed or you prefer to follow another documented method._

Login to your Synology using SSH and download the (latest JDK)[https://www.oracle.com/java/technologies/downloads/] (Oracle provides a stable URL for JDK which requires no login, so it's easier) from Oracle as following:

```bash
curl https://download.oracle.com/java/17/latest/jre-17_linux-x64_bin.tar.gz -o jre.tgz
```

Now, uncompress the JDK into `/usr/local` as following

```bash
sudo tar -xzf jdk.tgz -C /usr/local
```

If you wish, you can confirm the operation with the following command

```bash
sudo ls -ltd /usr/local/jdk*
```

I use a JDK 1.17 under `/usr/local` and I have a symlink to it. This is what you would see (I removed additional directories):

```bash
stefano@synology:~$ sudo ls -lt  /usr/local/
total 44
lrwxrwxrwx  1 root  root    12 Jun 13 09:28 java -> jdk-17.0.3.1
drwx------  9 root  root  4096 Jun 13 09:27 jdk-17.0.3.1
```

### Install the script

It's now time to install the script. Connect again to your Synology using SSH and run

```bash
sudo curl https://raw.githubusercontent.com/LunaticMuch/synology-jdownloader/develop/pkg-jdownloader.service \
-o /usr/local/lib/systemd/system/pkg-jdownloader.service
```

You should see the following:

```bash
sudo ls -l /usr/local/lib/systemd/system/pkg-jdownloader.service

-rw-r--r-- 1 root root  345 Jun 15 09:41 pkg-jdownloader.service
```

### Test the script

Run the following:

```bash
sudo systemctl start pkg-jdownloader.service
```

This will run the script. Now you can check modules in the kernel with `systemctl` as

```bash
root@synology:/volume1/@appstore/JDownloader# systemctl status pkg-jdownloader.service -l
● pkg-jdownloader.service - JDownloader
   Loaded: loaded (/usr/local/lib/systemd/system/pkg-jdownloader.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2022-06-15 09:43:40 BST; 7min ago
  Process: 16984 ExecStart=/volume1/@appstore/JDownloader/start.sh (code=exited, status=0/SUCCESS)
 Main PID: 16986 (java)
   Memory: 144.7M
   CGroup: /system.slice/pkg-jdownloader.service
           └─16986 java -jar JDownloader.jar
```

This means JDownloader is actually running

### Enable the script

The script must run at startup, so you need to tell `systemd` to do it

```bash
systemctl enable pkg-jdownloader.service
```

If you now reboot, this script will run before docker and the container will find the old ACM tty.

## Startup script (Source)

```bash
[Unit]
Description=JDownloader
After=syslog.target network.target

[Service]
SuccessExitStatus=143
Type=forking

WorkingDirectory=/volume1/@appstore/JDownloader
ExecStart=/volume1/@appstore/JDownloader/start.sh
TimeoutStartSec=3600
TimeoutStopSec=3600
RemainAfterExit=true
PIDFile=/volume1/@appstore/JDownloader/JDownloader.pid


[Install]
WantedBy=multi-user.target
```

## Limits

The script can accep additional parameters but it's not so flexible right not. `ExecStart` is required to run a `bash` script with the full path because this is the requirement of `systemd`. I plan to improve it, but for now it's fine and it's actually a script that you create and forget because it works.

## Disclaimer

USE THIS AT YOUR RISK
