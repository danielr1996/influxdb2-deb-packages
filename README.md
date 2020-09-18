# influxdb2-deb-packages

.deb packages for influxdb 2 beta relases

> This package doesn't follow best practices on debian packaging, instead it is just a simple package 
> to install influxdb 2 beta as a deb package. This package will be obsolete as soon as there are
> official deb releases for influxdb, therefor it is as simple as possible. This means:
> * no compilation from source, instead use the official influxdb tar.gz beta releases. 
> * for reduced complexity the binaries are directly included in the repository and may not be up to date
> * no sophisticed packageing process, just bunch of bash files

> This packages provides only arm64 builds because im using a raspberry pi. 
> But it should be relativly easy to build a package for other architectures by swapping the tarball url.

This package will 
* install the influx and influxd binaries
* install a systemd service file that runs `influxd` as root
* remove everything completely on uninstallation

This package will **not**
* provide any configuration, instead it will use the default configuration
* provide sources, man files or any additional stuff
* provide md5sums for the contained files
* define runtime dependencies (As there are no information on the official site), but it should work anyways because it basically just depends on libc, libgcc and some other which should be installed anyways.

Based on: https://docs.influxdata.com/influxdb/v2.0/get-started/#start-with-influxdb-oss and https://tldp.org/HOWTO/html_single/Debian-Binary-Package-Building-HOWTO/ 

## Usage
```
curl -s --compressed "https://danielr1996.github.io/influxdb2-deb-packages/key.gpg" | sudo apt-key add -
sudo curl -s --compressed -o /etc/apt/sources.list.d/influxdb2beta.list "https://danielr1996.github.io/influxdb2-deb-packages/repo.list"
sudo apt -y install influxdb
```

## Build
The binaries in this repository are for influxdb 2.0.0-beta.16 and obtained from the [InfluxDB site](hhttps://docs.influxdata.com/influxdb/v2.0/get-started/#optional-download-install-and-use-the-influx-cli)
See the next section for instructions on updating the version to package.

This repository includes a Vagrantfile to start a VM with the packages needed for packageing. To start the VM use
```shell script
vagrant up
vagrant ssh
```

Build the package with dpkg-deb and optionally lint it with lintian
```shell script
fakeroot dpkg-deb --build /vagrant/src /vagrant/docs
lintian /vagrant/docs/influx*.deb
```

The packaged deb file is now in the project root named influxdb_<version>_<arch>.deb.

## Updating the version
Download the newest release from the [InfluxDB site](hhttps://docs.influxdata.com/influxdb/v2.0/get-started/#optional-download-install-and-use-the-influx-cli), 
extract it and put influx and influxd binaries in src/usr/bin/

Update the version in src/DEBIAN/control

## Updating the package repo

Included in this repo is a github pages page in doc folder. If you built a new package and want to publish it use the following steps.
See https://assafmo.github.io/2019/05/02/ppa-repo-hosted-on-github.html for information on hosting a deb repo on github

```shell script
cd /vagrant/docs && dpkg-scanpackages --multiversion . > Packages && gzip -k -f Packages
apt-ftparchive release /vagrant/docs > /vagrant/docs/Release
gpg --default-key "danielrichter@posteo.de" -abs -o - /vagrant/docs/Release > /vagrant/docs/Release.gpg
gpg --default-key "danielrichter@posteo.de" --clearsign -o - /vagrant/docs/Release > /vagrant/docs/InRelease
```