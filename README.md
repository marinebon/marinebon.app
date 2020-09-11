# marinebon_app
docker for marinebon.app

Contents:
<!--
To update table of contents run: `cat README.md | ./gh-md-toc -`
Uses: https://github.com/ekalinin/github-markdown-toc
-->

* [Server software](#server-software)
* [Shell into Amazon EC2 server](#shell-into-amazon-ec2-server)
* [Install Docker](#install-docker)
   * [docker](#docker)
   * [docker-compose](#docker-compose)
* [Build containers](#build-containers)
   * [Test webserver](#test-webserver)
* [Setup domain marinebon.app](#setup-domain-mpatlas4rorg)
* [Run docker-compose](#run-docker-compose)
* [marinebon.app manual post-docker install steps](#mpatlas4r_org-manual-post-docker-install-steps)
* [Docker maintenance](#docker-maintenance)
   * [Push docker image](#push-docker-image)
   * [Develop on local host](#develop-on-local-host)
   * [Operate on all docker containers](#operate-on-all-docker-containers)
   * [Inspect docker logs](#inspect-docker-logs)
* [shiny app shuffle](#shiny-app-shuffle)
* [erddap quick fix](#erddap-quick-fix)
* [TODO](#todo)

## Server software

- Website:<br>
  **www.\***, **marinebon.app**
  - [Nginx](https://www.nginx.com/)
  - [Rmarkdown website](https://bookdown.org/yihui/rmarkdown/rmarkdown-site.html)
- Analytical apps:
  - [Shiny](https://shiny.rstudio.com)<br>
    **shiny.\***
  - [RStudio](https://rstudio.com/products/rstudio/#rstudio-server)<br>
    **rstudio.\***
- Spatial engine:
  - [GeoServer](http://geoserver.org)<br>
    **gs.\***
  - [PostGIS](https://postgis.net)<br>
    marinebon.app **:5432**

- Containerized using:
  - [docker](https://docs.docker.com/engine/installation/)
  - [docker-compose](https://docs.docker.com/compose/install/)
  - [nginx-proxy](https://github.com/jwilder/nginx-proxy)


## Create Server

Created droplet at https://digitalocean.com with ben@ecoquants.com (Google login):

- Choose an image : Distributions : Marketplace : 
  - **Docker** by DigitalOcean Version 19.03.12, OS Ubuntu 20.04
- Choose a plan : Basic : 
  - **$40 /mo** $0.060 /hour
  - 8 GB / 4 CPUs
  - 160 GB SSD disk
  - 5 TB transfer
- Choose a datacenter region :
  - **New York** 1
- Authentication :
  - **SSH key**
  - `ssh-keygen`:
    - Your identification has been saved in /Users/bbest/.ssh/id_rsa.
    - Your public key has been saved in /Users/bbest/.ssh/id_rsa.pub.
  - SSH key content: `cat ~/.ssh/id_rsa.pub`
    Name: macbook-pro-bbest
- How many Droplets?
  - **1  Droplet**
- Choose a hostname :
  - **marinebon.app**

Note IP address generated with new droplet, in this case: `164.90.217.11`.

## Setup domain marinebon.app

- Bought domain **marinebon.app** for **$12/yr** with account bdbest@gmail.com.

- DNS matched to server IP `164.90.217.11` to domain **marinebon.app** via [Google Domains]( https://domains.google.com/m/registrar/marinebon.app/dns), plus the following subdomains added under **Custom resource records** with:

- Type: **A**, Data:**164.90.217.11** and Name:
  - **@**
  - **api**
  - **erddap**
  - **geoserver**
  - **rstudio**
  - **shiny**
- Name: **www**, Type: **CNAME**, Data:**marinebon.app**

## Shell into server

```bash
ssh root@marinebon.app
```

Before using secure shell (SSH), you'll need to the public key of your machine and username added to the server. See [How To Set Up SSH Keys | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2).

The first time you login from a machine, you'll need to type `yes`:

```
The authenticity of host '192.241.142.34 (192.241.142.34)' can't be established.
ECDSA key fingerprint is SHA256:XisfpVW6doSV9WDZH5sHAcDpRK3LR3AQF2rNdfUeDOA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.241.142.34' (ECDSA) to the list of known hosts.
```

## Check Docker

Confirm that `docker` and `docker-compose` are installed:

```
docker --version
# Docker version 19.03.12, build 48a66213fe

docker-compose --version
# docker-compose version 1.22.0, build f46880fe

systemctl status docker
```

References:

- [How To Install and Use Docker on Ubuntu 18.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)
- [Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/)


## Build containers

### Test webserver

Reference:

- [How To Run Nginx in a Docker Container on Ubuntu 14.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-nginx-in-a-docker-container-on-ubuntu-14-04)

```bash
docker run --name test-web -p 80:80 -d nginx

# confirm working
docker ps
curl http://localhost
```

returns:
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

You can also visit http://164.90.217.11 to see this in your browser.

Turn off:

```bash
docker stop test-web
```

## Run docker-compose

References:

- [Quickstart: Compose and WordPress | Docker Documentation](https://docs.docker.com/compose/wordpress/)
- [docker-compose.yml · kartoza/docker-geoserver](https://github.com/kartoza/docker-geoserver/blob/master/docker-compose.yml)
- [How To Install WordPress With Docker Compose | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-docker-compose)


First, you will create the environment `.env` file to specify password and host:

- NOTE: Set `PASSWORD`, substituting "S3cr!tpw" with your password. The [docker-compose.yml](https://github.com/BenioffOceanInitiative/s4w-docker/blob/master/docker-compose.yml) uses [variable substitution in Docker](https://docs.docker.com/compose/compose-file/#variable-substitution).

```bash
# get latest docker-compose files
git clone https://github.com/marinebon/marinebon_app.git
cd marinebon_app

# set environment variables
echo 'PASSWORD=S3cr!tpw' > .env
echo 'HOST=marinebon.app' >> .env
cat .env

# launch
docker-compose up -d

# OR update
git pull; docker-compose up -d

# OR build if Dockerfile updated in subfolder
git pull; docker-compose up --build -d

# OR restart
docker-compose restart

# OR stop
docker-compose stop
```

## Docker maintenance

### Push docker image

This image could be pushed as a custom image, eg `bdbest/rstudio-shiny:s4w`, I [docker-compose push](https://docs.docker.com/compose/reference/push/) to [bdbest/rstudio-shiny:s4w | Docker Hub](https://hub.docker.com/layers/bdbest/rstudio-shiny/s4w/images/sha256-134b85760fc6f383309e71490be99b8a50ab1db6b0bc864861f9341bf6517eca).

```bash
# login to docker hub
docker login --username=bdbest

# push updated image
docker-compose push
```

### Develop on local host

Note setting of `HOST` to `local` vs `marinebon.app`:

```bash
# get latest docker-compose files
git clone https://github.com/marinebon/marinebon_app.git
cd ~/marinebon_app

# set environment variables
echo "PASSWORD=S3cr!tpw" > .env
echo "HOST=local" >> .env
cat .env

# launch
docker-compose up -d

# see all containers
docker ps -a
```

Then visit http://localhost or http://rstudio.localhost.

TODO: try migrating volumes in /var/lib/docker onto local machine.

### Operate on all docker containers

```bash
# stop all running containers
docker stop $(docker ps -q)

# remove all containers
docker rm $(docker ps -aq)

# remove all image
docker rmi $(docker images -q)

# determine how volumes used
docker ps -a --filter volume=[volumename]

# remove all volumes
docker volume rm $(docker volume ls -q)

# remove all stopped containers
docker container prune
```

### Inspect docker logs

To tail the logs from the Docker containers in realtime, run:

```bash
docker-compose logs -f

docker inspect rstudio-shiny
```

## Check ogr2ogr on rstudio 

Since using docker postgis/postgis 12.3, try on rstudio Terminal:

```bash
sudo apt-get update
sudo apt-get install gdal-bin
ogrinfo --version
# GDAL 2.4.0, released 2018/12/14
```

### Load db

```bash
passwd=`cat /share/config/marinebon.app_pass.txt`
echo $passwd
ogrinfo -ro PG:"host=postgis dbname=gis user=admin password=$passwd"

ogr2ogr -f "PostgreSQL" PG:"host=postgis dbname=gis user=admin password=$passwd" "/share/mpatlas_mpa.gpkg" -nln "mpa_mpa" # -append
```

## Post-Docker build steps

In https://rstudio.marinebon.app Terminal as admin:

```bash
sudo chown -R admin /share

# git clone private repos
cd /share/github
git clone https://github.com/marinebon/www_marinebon.git

# link special folders to admin home folder
ln -s /srv/shiny-server     /home/admin/shiny_apps
ln -s /var/log/shiny-server /home/admin/shiny_logs
ln -s /share                /home/admin/share
ln -s /share/github         /home/admin/github
```

Upload `marinebon.app_pass.txt` into `/share/config/`.


