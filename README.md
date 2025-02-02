#### Table of Contents

1. [About](#about)
1. [Requirements](#requirements)
1. [Provision Instructions](#provision-instructions)
1. [Build Instructions](#build-instructions)
1. [Acknowledgement](#acknowledgement)

## About

This repository contains everything needed to build a customized [mfsBSD](https://mfsbsd.vx.sk/) image that can be used to provision [FreeBSD](https://www.freebsd.org/) servers with [Foreman](https://theforeman.org/).

## Requirements

In order to deploy FreeBSD servers with Foreman, the following requirements must be met:

* Provisioning templates for FreeBSD are already [included](https://github.com/theforeman/foreman/blob/develop/app/views/unattended/provisioning_templates/provision/freebsd_(mfsbsd)_provision.erb) in Foreman
* A custom mfsBSD image, can be build manually (see below) or [downloaded from here](https://github.com/fraenki/freebsd-foreman/releases)
* A working PXE/DHCP/TFTP boot environment (Foreman Smart Proxy)
* Server or VM needs at least 1 GB of RAM to store the image during installation

## Provision Instructions

* Create a "installation media" entry in Foreman for FreeBSD (use local or official mirror, i.e. `http://ftp.freebsd.org/pub/FreeBSD/releases/$arch/$major.$minor-RELEASE/`)
* Create a new "operating system" entry in Foreman for the desired FreeBSD release
* Copy the custom mfsBSD image to your TFTP server
* Create a new host in Foreman, watch the FreeBSD installation begin :)

## Build Instructions

Use `build.sh` to create your own images or to include a modified version of `rc.local`. The script must be run as root (on a FreeBSD server).

```
# git clone https://github.com/fraenki/freebsd-foreman.git
# cd freebsd-foreman
# ./build.sh -r 13.1
```

## Acknowledgement

Thanks to Martin Matuška for creating [mfsBSD](https://mfsbsd.vx.sk/) and his talk on [Deploying FreeBSD systems with Foreman](https://blog.vx.sk/archives/60).

## Extra: Previous steps
```
mkdir -p /workdir/DIST  
cd /workdir
fetch -4 -q -o "mfsbsd-master.tar.gz" --no-verify-peer "https://github.com/mmatuska/mfsbsd/archive/master.tar.gz"
fetch -o DIST ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/13.2-RELEASE/kernel.txz
fetch -o DIST ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/13.2-RELEASE/base.txz
tar xf mfsbsd-master.tar.gz
chown -R root:wheel mfsbsd-master
cd mfsbsd-master
make BASE=/workdir/DIST RELEASE=13.2-RELEASE ARCH=amd64 PKG_STATIC=/usr/local/sbin/pkg-static MFSROOT_MAXSIZE=120m
#move the image to /tmp/image
git clone https://github.com/fraenki/freebsd-foreman.git
cd freebsd-foreman
#Modify the build.sh script not to exit if image not fetched
./build.sh -r 13.2
scp /tmp/image/FreeBSD-x86_64-13.2-mfs.img X.X.X.X:/var/lib/tftpboot/boot/.
```

## Next Step: OS Version Error
To fix, this you need to re-bootstrap pkg as follows:
```
root@Inst1:/usr/home/neel # pkg-static bootstrap -f
pkg(8) is already installed. Forcing reinstallation through pkg(7).
The package management tool is not yet installed on your system.
Do you want to fetch and install it now? [y/N]: y
Bootstrapping pkg from pkg+http://pkg.FreeBSD.org/FreeBSD:12:amd64/quarterly, please wait...
Verifying signature with trusted certificate pkg.freebsd.org.2013102301... done
Installing pkg-1.13.2...
package pkg is already installed, forced install
Extracting pkg-1.13.2: 100%
root@Inst1:/usr/home/neel #
```
And then update as normal:
```
root@Inst1:/usr/home/neel # pkg update -f
Updating FreeBSD repository catalogue...
Fetching meta.conf: 100%    163 B   0.2kB/s    00:01    
Fetching packagesite.txz: 100%    6 MiB   3.3MB/s    00:02    
Processing entries: 100%
FreeBSD repository update completed. 31517 packages processed.
All repositories are up to date.
root@Inst1:/usr/home/neel # pkg upgrade
Updaing FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
```


