# JDownloader configuration for DSM7

This document helps with the setup of a startup script for running (JDownloader)[https://jdownloader.org] on Synology with DSM7

## Intro

DSM7 uses `systemd` for services statup operations. Systemd allows additional scripts and complex chaining of these. While systemd can be irksume manytimes, it's quite powerful and it is better to use it. This ensures any future update of DSM7 won't break your script and the startup.

## Usage

### A note on my environment

This script is actually based on my personal Synolody DSM7 setup. 

I use a JDK 1.8 under `/usr/local` and I have a symlink to it. This is what you would see (I removed additional directories):

```bash
stefano@synology:~$ sudo ls -lt  /usr/local/ 
total 44
lrwxrwxrwx  1 root  root    12 Jun 13 09:28 java -> jdk-18.0.1.1
drwx------  9 root  root  4096 Jun 13 09:27 jdk-18.0.1.1
```

### Prepare the script

Login via SSH into you synology. Create/Copy/Move the startup script into `/usr/local/lib/systemd/system`

You should see the following:

```bash
ls -l /usr/local/lib/systemd/system/pkg-jdownloader.service 

-rwx--x--x 1 root root 341 Jun 13 11:32 /usr/local/lib/systemd/system/pkg-jdownloader.service
```

### Test the script

Run the following:

```bash
sudo systemctl start pkg-jdownloader.service
```

This will run the script. Now you can check modules in the kernel with `systemctl` as

```bash
stefano@synology:~$ sudo systemctl status pkg-jdownloader.service
● pkg-jdownloader.service - JDownloader
   Loaded: loaded (/usr/local/lib/systemd/system/pkg-jdownloader.service; disabled; vendor preset: disabled)
   Active: active (exited) since Mon 2022-06-13 11:32:48 BST; 4h 45min ago
  Process: 27062 ExecStart=/bin/bash -c /usr/local/jdk-18.0.1.1/bin/java -jar JDownloader.jar (code=exited, status=0/SUCCESS)
 Main PID: 27062 (code=exited, status=0/SUCCESS)
   Memory: 6.0G
   CGroup: /system.slice/pkg-jdownloader.service
           └─20701 /usr/local/jdk-18.0.1.1/bin/java -jar JDownloader.jar -afterupdate
```

This means JDownloader is actually running

### Enable the script

The script must run at startup, so you need to tell `systemd` to do it

```bash
systemctl enable pkg-jdownloader.service
```

If you now reboot, this script will run before docker and the container will find the old ACM tty.


##  Startup script

```bash
[Unit]
Description=JDownloader
After=syslog.target network.target

[Service]
SuccessExitStatus=143
Type=simple

WorkingDirectory=/volume1/@appstore/JDownloader
ExecStart=/bin/bash -c "/usr/local/jdk-18.0.1.1/bin/java -jar JDownloader.jar" 
TimeoutStartSec=3600
TimeoutStopSec=3600
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

## Limits

The script can accep additional parameters but it's not so flexible right not. `ExecStart` is required to run a `bash` script with the full path because this is the requirement of `systemd`. I plan to improve it, but for now it's fine and it's actually a script that you create and forget because it works.

## Disclaimer

USE THIS AT YOUR RISK