![Alt text](http://i.imgur.com/InPPMtr.png "")

- [Introduction](#introduction)
  - [Contributing](#contributing)
  - [Issues](#issues)
    - [Bugs](#bugs)
- [Getting started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
    - [Docker Hub](#docker-hub)
    - [GitHub](#github)
    - [Initial Configuration](#initial-configuration)
- [Maintenance](#maintenance)
  - [Upgrading](#upgrading)
  - [Automatic Upgrades](#automatic-upgrades)
  - [Removal](#removal)
  - [Shell Access](#shell-access)
- [unRAID](#unraid)
  - [Installation](#unraid-installation)
  - [Automatic Upgrades](#unraid-automatic-upgrades)
- [Technical Information](#technical-information)
  - [Environment Variables](#environment-variables)
  - [Volumes](#volumes)
- [Manual Run and Installation](#manual-run-and-installation)
- [License](#license)
- [Donation](#donation)


# Introduction

Ubooquity is a free, lightweight and easy-to-use home server for your comics
and ebooks. Use it to access your files from anywhere, with a tablet, an
e-reader, a phone or a computer. Because Ubooquity was written in Java, it may
be run on any operating system with Java support.

This subfolder contains all necessary files to build a [Docker](https://www.docker.com/) image for [Ubooquity](www.ubooquity.org).

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D

## Issues

Before reporting your issue please try updating Docker to the latest version
and check if it resolves the issue. Refer to the Docker [installation guide](https://docs.docker.com/installation) for instructions.

SELinux users should try disabling SELinux using the command `setenforce 0` to see if it resolves the issue.

If the above recommendations do not help then [report your issue](../../issues/new) along with the following information:

- Output of the `docker version` and `docker info` commands
- The `docker run` command or `docker-compose.yml` used to start the image. Mask out the sensitive bits.
- Please state if you are using [Boot2Docker](http://www.boot2docker.io), [VirtualBox](https://www.virtualbox.org), etc.

### Bugs

- Ubooquity on start complains about not being able extract fonts

# Getting started

## Prerequisites

* ensure the user running the installation command can run docker

## Installation:

### [Docker Hub](https://hub.docker.com/r/hurricane/ubooquity/):
It is recommended you install directly from the [Docker Hub](https://hub.docker.com/r/hurricane/ubooquity/).
Before starting the install procedure please verify that any and all
prerequisites are fulfilled:

The following command copies over a wrapper scrip that will create a container
named `ubooquity` when executed. The wrapper script will ensure that the
container gets setup with the appropriate environment variables and volumes
each time it is executed. It does so by prompting and saving certain setting to
answer file that it will look up each time it is called.

Start the installation by issuing the following command from within a terminal:
```
docker run -it --rm -v /usr/local/bin:/target \
    hurricane/ubooquity instl
```

Optionally, you can also install a systemd service file. Before installing the
systemd service file, you might want specify which user you wish the daemon to
run as, specifically if it differs from the user running the installation. You
can do this by reinstalling ubooquity with the following command:
```
docker run -it --rm -v /usr/local/bin:/target -e "APP_USER=username" \
    hurricane/ubooquity instl
```

If the user you specified does not have a valid home directory you will
probably want to specify an alternate location to store the ubooquity
configuration and database like so:
```
docker run -it --rm -v /usr/local/bin:/target \
    -e "APP_USER=username" \
    -e "APP_CONFIG=/var/lib/ubooquity" \
    hurricane/ubooquity instl
```
Above, change the `username` to the name of the user you wish to run the daemon
as, and adjust `/var/lib/ubooquity` to wherever it is you wish to store your
ubooquity database and information. Please verify that `$APP_USER` is the owner
of `$APP_CONFIG`.

Afterward, proceed with the service file installation:
```
docker run -it --rm -v /etc/systemd/system:/target \
   hurricane/ubooquity instl service
```

If you installed the systemd service file, you can enable Ubooquity to
automatically start when the system boots by executing the following command:
```
sudo systemctl enable ubooquity.service
```

### [GitHub](https://github.com/hurricane/docker-containers/ubooquity):
Installation from GitHub is recommended only for the purposes of
troubleshooting and development. To install Ubooquity from GitHub execute the
following:
```
git clone https://github.com/hurricane/docker-containers
cd docker-containers/ubooquity
make instl
```

Additionally, you can install the systemd service file after executing the
above by issuing the following:
```
make service
```

### Initial Configuration:

Once Ubooquity has been installed you can simply execute the binary from a terminal:
```
ubooquity
```

The first time you run the Ubooquity container it will prompt you for the
location of your ebook library. Enter one response per query.  This will ensure
that the container gets access to the host's file system from within the
containerized environment.


# Maintenance

## Upgrading:

If you have installed our systemd service file, you can update by
executing the following command:
```
sudo systemctl restart ubooquity.service
```

Additionally you can update by:
```
docker exec ubooquity update
```

Or by executing:
```
docker pull hurricane/ubooquity
docker stop ubooquity
ubooquity
```

## Automatic Upgrades:

In order to have the container periodically check and upgrade the ubooquity
binary one needs to add  a [`crontab`](https://en.wikipedia.org/wiki/Cron)
entry. Like so:
```
echo "0 2 * * * docker exec ubooquity update" | sudo tee -a /var/spool/cron/crontabs/root
```

## Removal:

```bash
docker run -it --rm \
  --volume /usr/local/bin:/target \
  hurricane/ubooquity uninstl
```

## Shell Access

For debugging and maintenance purposes you may want access the containers
shell. If you are using Docker version `1.3.0` or higher you can access
a running containers shell by starting `bash` using `docker exec`:

```bash
docker exec -it ubooquity bash
```


# unRAID:

You can find the template for this container on GitHub. Located [here](https://github.com/hurricanehrndz/container-templates/tree/master/hurricane).

## unRAID Installation:

Please navigate to the Docker settings page on unRAID's Web-UI and under repositories add:
```
https://github.com/hurricanehrndz/container-templates/tree/master/hurricane
```
For more information on adding templates to unRAID please visit the [unRAID forums](https://lime-technology.com/forum/).

## unRAID Automatic Upgrades:

On unRAID, execute the following to have the container periodically update
itself. Additionally, add the same line of code to your `go` file to make the
change persistent.
```
echo "0 2 * * * docker exec Ubooquity update" | sudo tee -a /var/spool/cron/crontabs/root
```


# Technical information:

By default the image has been created to run with UID and GID 1000. If using
the automatic install method from Docker, the container is set to run with the
UID and GID of of the user executing the `ubooquity` wrapper script.
Additionally, the wrapper script by default saves ubooquity's database and
settings in a hidden sub folder in the executing user's home directory. Most
default settings can be adjusted by passing the appropriate environment
variable. Here is a list of any and all applicable environment variables that
can be override by the end user.

## Environment Variables:

You may overwrite the default settings by passing the appropriate environment variable:
* APP_USER   - name of user to create within container for purposes of running ubooquity, UID, GID are more important.
* APP_UID    - UID assigned to APP_USER upon creation.
* APP_GID    - GID assigned to APP_USER upon creation.
* APP_CONFIG - the directory that should house Ubooquity metadata and configuration (folder on host)
* MAX_MEM    - maximum Java heap size in megabytes. Default value is 512.

Please read Docker documentation on [environment variables](https://docs.docker.com/engine/reference/run/#env-environment-variables) for more information.

## Volumes:

These are folders of interest on the guest's filesystem.

* `/config`  - Folder to store Ubooquity's log, configuration and database.


# Manual Run and Installation:

Of course you can always run docker image manually. Please be aware that if you
wish your data to remain persistent you need to provide a location for the
`/config` volume. For example,
```
docker run -d --net=host -v /*your_ubooquityhome_location*:/config \
                         -e TZ=America/Edmonton
                         --name=ubooquity hurricane/ubooquity
```
All the information mention previously regarding user UID and GID still applies
when executing a docker run command.


# License

Code released under the [MIT license](./LICENSE).


# Donation

[@hurricanehrndz](https://github.com/hurricanehrndz): [![PayPal](https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=74S5RK533DD6C)
