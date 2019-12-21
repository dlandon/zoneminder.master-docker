## Zoneminder Docker
(Current version: 1.33 (master))

### About
This is an easy to run dockerized image of [ZoneMinder](https://github.com/ZoneMinder/zoneminder) along with the the [ZM Event Notification Server](https://github.com/pliablepixels/zmeventnotification) and its machine learning subsystem (which is disabled by default but can be enabled by a simple configuration).  

The configuration settings that are needed for this implementation of Zoneminder are pre-applied and do not need to be changed on the first run of Zoneminder.

This verson will now upgrade from previous versions.

### Installation
Install the docker by going to a command line and enter the command:

```bash
docker pull dlandon/zoneminder
```

This will pull the zoneminder docker image.  Once it is installed you are ready to run the docker.

Before you run the image, feel free to read configuration section below to customize various settings

To run Zoneminder:

```bash
docker run -d --name="Zoneminder" \
--net="bridge" \
--privileged="true" \
-p 8443:443/tcp \
-p 9000:9000/tcp \
-e TZ="America/New_York" \
-e SHMEM="50%" \
-e PUID="99" \
-e PGID="100" \
-e INSTALL_HOOK="0" \
-e INSTALL_FACE="0" \
-e INSTALL_TINY_YOLO="0" \
-e INSTALL_YOLO="0" \
-v "/mnt/Zoneminder":"/config":rw \
-v "/mnt/Zoneminder/data":"/var/cache/zoneminder":rw \
dlandon/zoneminder
```

For http:// access use: -p 8080:80/tcp

**Note**: If you have opted to install face recognition, and/or have opted to download the yolo models, it takes time.
Face recognition in particular can take several minutes (or more). Once the `docker run` command above completes, you may not be able to access ZoneMinder till all the downloads are done. To follow along the installation progress, do a `docker logs -f zoneminder` to see the syslog for the container that was created above.

### Subsequent runs

You can start/stop/restart the container anytime. You don't need to run the command above every time. If you have already created the container once (by the `docker run` command above), you can simply do a `docker stop Zoneminder` to stop it and a `docker start Zoneminder` to start it anytime (or do a `docker restart zoneminder`)

#### Customization

- Set `INSTALL_HOOK="1"` to install the hook processing packages and run setup.py to prepare the hook processing.  The initial installation can take a long time.
- Set `INSTALL_FACE="1"` to install face recognition packages.  The initial installation can take a long time.
- Set `INSTALL_TINY_YOLO="1"` to install the tiny yolo hook processing files.
- Set `INSTALL_YOLO="1"` to install the yolo hook processing files.
- The command above use a host path of `/mnt/Zoneminder` to map the container config and cache directories. This is going to be persistent directory that will retain data across container/image stop/restart/deletes. ZM mysql/other config data/event files/etc are kept here. You can change this to any directory in your host path that you want to.

#### Post install configuration and caveats

- After successful installation, please refer to the [ZoneMinder](https://zoneminder.readthedocs.io/en/stable/), [Event Server and Machine Learning](https://zmeventnotification.readthedocs.io/en/latest/index.html) configuration guides from the authors of these components to set it up to your needs. Specifically, if you are using the Event Server and the Machine learning hooks, you will need to customize `/etc/zm/zmeventnotification.ini` and `/etc/zm/objectconfig.ini`

- Note that by default, this docker build runs ZM on port 443 inside the docker container and maps it to port 8443 for the outside world. Therefore, if you are configuring `/etc/zm/objectconfig.ini` or `/etc/zm/zmeventnotification.ini` remember to use `https://localhost:443/<etc>` as the base URL

- Push notifications with images will not work unless you replace the self-signed certificates that are auto-generated. Feel free to use the excellent and free [LetsEncrypt](https://letsencrypt.org) service if you'd like.

#### Usage

To access the Zoneminder gui, browse to: `https://<your host ip>:8443/zm`

The zmNinja Event Notification Server is accessed at port `9000`.  Security with a self signed certificate is enabled.  You may have to install the certificate on iOS devices for the event notification to work properly.

#### Change Log

2019-12-02
- Update zmNinja Event Notification Server to version 5.2.

2019-12-01
- Fix ES file references to newer versions of zm_detect_wrapper.sh amnd zm_detect.py

2019-11-26
- Remove set_php_time_zone script.  The time is now set in options.

2019-11-23
- Update zmNinja Event Notification Server to version 4.6.

2019-11-09
- Update zmNinja Event Notification Server to version 4.5.

2019-11-06
- Add libavutil-dev for hwaccel support.

2019-10-26
- Remove cambozola legacy browser support.

2019-10-05
- Update zmNinja Event Notification Server to version 4.4.

2019-09-22
- Upgrade to php 7.3.

2019-09-21
- Fix /hook folder permission check.

2019-09-19
- Update zmes_hook_helpers.

2019-09-16
- Initial master release.
