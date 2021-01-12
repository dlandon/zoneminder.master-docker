## Zoneminder Docker
(Current version: 1.35 (master))

We have set up a GoFundMe to fund the development of a new Docker that will be for Zoneminder and ES/ML with all the ES/ML modules pre-configured and for maintenance and support going forward.

[GoFundMe](https://www.gofundme.com/f/maintenance-of-zoneminder-docker-with-es-and-ml?utm_source=customer&utm_medium=copy_link&utm_campaign=p_cf+share-flow-1)

### About
This is an easy to run dockerized image of [ZoneMinder](https://github.com/ZoneMinder/zoneminder) along with the the [ZM Event Notification Server](https://github.com/pliablepixels/zmeventnotification) and its machine learning subsystem (which is disabled by default but can be enabled by a simple configuration).  

The configuration settings that are needed for this implementation of Zoneminder are pre-applied and do not need to be changed on the first run of Zoneminder.

This verson will now upgrade from previous versions.

### Installation
Install the docker by going to a command line and enter the command:

```bash
docker pull dlandon/zoneminder.master
```

This will pull the zoneminder master docker image. Note that ZoneMinder master should always be treated as a development release. If you want to run the latest stable release, please use [this](https://github.com/dlandon/zoneminder) repo. Once it is installed you are ready to run the docker.

Before you run the image, feel free to read configuration section below to customize various settings

To run Zoneminder:

```bash
docker run -d --name="Zoneminder" \
--net="bridge" \
--privileged="false" \
--shm-size="5G" \
-p 8443:443/tcp \
-p 9000:9000/tcp \
-e TZ="America/New_York" \
-e PUID="99" \
-e PGID="100" \
-e INSTALL_HOOK="0" \
-e INSTALL_FACE="0" \
-e INSTALL_TINY_YOLOV3="0" \
-e INSTALL_TINY_YOLOV4="0" \
-e INSTALL_YOLOV3="0" \
-e INSTALL_YOLOV4="0" \
-v "/mnt/Zoneminder":"/config":rw \
-v "/mnt/Zoneminder/data":"/var/cache/zoneminder":rw \
dlandon/zoneminder.master
```

For http:// access use: -p 8080:80/tcp

### Shared Memory
Set your shared memory to half of your installed memory.

**Note**: If you have opted to install face recognition, and/or have opted to download the yolo models, it takes time.
Face recognition in particular can take several minutes (or more). Once the `docker run` command above completes, you may not be able to access ZoneMinder till all the downloads are done. To follow along the installation progress, do a `docker logs -f zoneminder` to see the syslog for the container that was created above.

### Subsequent runs

You can start/stop/restart the container anytime. You don't need to run the command above every time. If you have already created the container once (by the `docker run` command above), you can simply do a `docker stop Zoneminder` to stop it and a `docker start Zoneminder` to start it anytime (or do a `docker restart zoneminder`)

#### Customization

- Set `INSTALL_HOOK="1"` to install the hook processing packages and run setup.py to prepare the hook processing.  The initial installation can take a long time.
- Set `INSTALL_FACE="1"` to install face recognition packages.  The initial installation can take a long time.
- Set `INSTALL_TINY_YOLOV3="1"` to install the tiny yolo v3 hook processing files.
- Set `INSTALL_TINY_YOLOV4="1"` to install the tiny yolo v4 hook processing files.
- Set `INSTALL_YOLOV3="1"` to install the yolo hook v3 processing files.
- Set `INSTALL_YOLOV4="1"` to install the yolo hook v4 processing files.
- The command above use a host path of `/mnt/Zoneminder` to map the container config and cache directories. This is going to be persistent directory that will retain data across container/image stop/restart/deletes. ZM mysql/other config data/event files/etc are kept here. You can change this to any directory in your host path that you want to.

#### Post install configuration and caveats

- After successful installation, please refer to the [ZoneMinder](https://zoneminder.readthedocs.io/en/stable/), [Event Server and Machine Learning](https://zmeventnotification.readthedocs.io/en/latest/index.html) configuration guides from the authors of these components to set it up to your needs. Specifically, if you are using the Event Server and the Machine learning hooks, you will need to customize `/etc/zm/zmeventnotification.ini` and `/etc/zm/objectconfig.ini`

- Note that by default, this docker build runs ZM on port 443 inside the docker container and maps it to port 8443 for the outside world. Therefore, if you are configuring `/etc/zm/objectconfig.ini` or `/etc/zm/zmeventnotification.ini` remember to use `https://localhost:443/<etc>` as the base URL

- Push notifications with images will not work unless you replace the self-signed certificates that are auto-generated. Feel free to use the excellent and free [LetsEncrypt](https://letsencrypt.org) service if you'd like.

#### Usage

To access the Zoneminder gui, browse to: `https://<your host ip>:8443/zm`

The zmNinja Event Notification Server is accessed at port `9000`.  Security with a self signed certificate is enabled.  You may have to install the certificate on iOS devices for the event notification to work properly.

#### Troubleshooting when the docker fails

If you have a situation where the docker fails to start, you can set an environemtnt variable when the docker is started and MySql and Zoneminder will not be started.  This will keep the docker running so you can get into a command line in the docker and troubleshoot the problem.

Create an environment variable:
NO_START_ZM="1"

MySql and Zoneminder will not be started.

Get into a command line in the docker and troubleshoot your issue by using the following commands to start MySql and zonemonder and fix any errors/problems with them starting.

service mysql start

service zoneminder start
