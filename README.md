# Build a SDI with Docker

## Introduction

### Why Docker?
Docker is the de-facto standard for building image based containers and micro-service based infrastructures. It is
stable, has a big community and is easy-to-use. Docker has many potential use-cases. You can use it just as
a tool to try commands in different operating systems. It is easy to provision and to enhance and easy to clean-up.
You can run deeply specified tasks in a complete independent environment. You can build a service platform in
micro-service pattern. Furthermore it keeps logic in images and data in volumes clearly divided.

### License
There are two different versions of Docker. The community-edition is for free and under Apache 2.0 license.
For the enterprise edition you have to pay a fee. 

### Prerequirements and installation
#### Linux

For installing Docker on Linux you have to do these steps on a Debian based system:

```bash
curl -fsSL get.docker.com | sudo sh
```

**`docker-compose`**

Only on Linux `docker-compose` is not installed by default, but the installation is quite easy. Just download
the python script and make it executable with these commands as root user:

```bash
curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` \
  -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```
#### MacOS

Just follow the instruction for installing Docker Community Edition for MacOS on the Docker website. You should get a
`*.dmg` file that you have to mount with a double-click. After that just paste the Docker Application into your
system-wide application folder. Go into this application folder and run the Docker app. Now you can run the Docker
command on your terminal.

### Getting help and images

Docker has a big community. A lot of answers you will find in StackOverflow. There are also a lot of talks on YouTube
for example from the conference for Docker called "DockerCon". The project is growing very fast, this is why you should
follow online resources for docker.

#### Command line interface (CLI)

The Docker commandline interface is the best place to get quick help. If you want to get additional information on a 
command you can prefix it with `--help`. The printed messages are very useful!

#### Docker Store

To explore already build images visit: [`https://hub.docker.com`](https://hub.docker.com). Here you can also create an
account and publish your own images. Please choose images from the Docker Hub or Docker Store very carefully! Prefer
automated builds, because they are reproducible. Prefer also official namespaces.

#### Other resources of this workshop

This workshop is published on GitHub on [`https://github.com/yagajs/workshop-docker-geo-basics-booklet`](https://github.com/yagajs/workshop-docker-geo-basics-booklet).
If you have an offline version take a look for updates at the online resources.

## Getting started with Docker

At first simply call the following command in your terminal:

```bash
docker
```
The Docker commandline interface (CLI) has very useful help messages.

In our first exercise We want to run a container with Docker. A container is comparable to a virtual machine in a
virtualization environment. To start a virtual machine we need an image for it, in Docker we need it also. A big
advantage of Docker is, they have a big community-driven registry of ready-to-use Docker images.

To start a container initially we have to use the command `docker run`. You can also try this command in your
terminal, it will give you a short usage information. To get a complete list of parameters just run `docker run --help`.

### Say hello with docker

Now, more details on our first exercise. We want to run a typical "Hello World!" example container. For this purpose
we use an image of the registry to send greetings with the Docker whale called whalesay.

Let's try it with the following command:

```bash
docker run --rm docker/whalesay cowsay Hello YAGA
```

This is the anatomy of this command, for getting a better understanding for that command.

1. `docker`: The application on our operating system, of course.
2. `run`: Our command for the docker-cli to start a new container as described above.
3. `--rm`: A parameter. You can add as much parameters as you want, but only at this place, that means not before or
after this step. The `--rm` parameter causes a deletion of the container right after it has finished. Otherwise it will
stop and persists.
4. `docker/whalesay`: This is the name of the image we want to run. At first the Docker CLI tries to run the image that
you have on your system. If it can't find one, it will request that image from the global registry. The first part of
this name is for the namespace of the image. **You should always make a well considered decision before trusting a
foreign namespace!**. The second part after the `/` is the name of the image. In our case we use the `whalesay` image
of the official `docker` namespace.
5. `cowsay`: This is the command we want to run in our container. This value is optional, because every image has a
default command. This part of the anatomy is similar to part one, with one difference: it will run in our newly created
container.
6. `Hello Yaga`: Parameters for the command in step five. This value is optional, too.

You can replace the content for part five and six with other known bash commands, let's try:

```bash
docker run --rm docker/whalesay pwd
docker run --rm docker/whalesay ls -l
```

In the interactive mode you can even use shells like `bash` in a container:

```bash
docker run --rm -it docker/whalesay bash
```

Now you can work on the created container in a shell session. Try the following command:

```bash
cowsay Hello from inside a docker-container
```

With `Ctrl-D` or the `exit` command you can quit the session, as usual.

### Simply switch between operating systems

Say, you want to try the installation of a software on different operating-systems. With Docker we have the advantage
to simply switch on a different and clean operating system and try to install a software like `curl`.

#### Debian

```bash
docker run --rm -it debian
```

*Official images do not have to have a namespace. In addition bare images of operating systems, like Debian, uses its
shell as default command on the container. This is why there is no need to postfix the complete command with `/bin/bash`
to run it.*

Now install curl in your container and fetch a website with it:

```bash
apt-get update
apt-get install -y curl
curl https:/yagajs.org
```

### Alpine

Alpine is predestined for using it in containers, because it is build for such a reason. The installation of software in
Alpine is very easy and quick. It avoid creating temporary data and can stage software packages. The biggest advantage
in your daily business with Alpine is the size. It will need less then 5 MB, instead of more then 100 MB for Ubuntu or
Debian. This will accelerate your deployment-process extremely and can also cause better performance.

Start your Alpine terminal with:

```bash
docker run --rm -it alpine
```

Now install curl in your Alpine container and fetch a website with it, like we did before on Debian:

```bash
apk add --no-cache curl
curl https:/yagajs.org
```

### Mount local directories

You can mount a local file or folder on your container with the `-v` parameter. This parameter have the following
synopsis for mounting files and directories from host system ro containers:
`-v /absolute/path/on/host:/absolute/path/on/container`.

```bash
cd /tmp # Let's not pollute our host system...
docker run --rm -it -v $PWD:/my-pwd-on-host alpine
```

*Note: You must always give an absolute path to share a resource from your local filesystem. You can simply generate
relative paths by using the environment variable for your present working directory (`$PWD/relative/path/to/resource`).*

Now you can work with the directory of your host system on your container. They will be in sync all the time. Let's
inspect and write a little bit:

```bash
cd /my-pwd-on-host
mkdir from-docker
cd from-docker
echo "This file was created by Alpine running on Docker..." > info.txt
```

Now check it out on your host filesystem. You should find an `info.txt` file in the `from-docker` directory.

### Create a static HTML server

At first just run a simple HTTP server:

```bash
docker run --rm -p 8080:80 httpd
```

*You can terminate a Docker container like a running process on your local host system, by pressing `Ctrl-C`.*

Open your web-server in your browser of choice on the following address: `http://localhost:8080`. With the `-p`
parameter you can bind a network port of the container on your host system. It has the pattern
`(port-on-host-system):(port-in-container)`. So in our case we bind the port `80` of the container to port `8080` on our
local host operating system. You can also enhance it with a listener address with this pattern:
`(address-to-listen):(port-on-host-system):(port-in-container)`.

Let's mount our already created data in our base directory on our web-server and publish it this way on HTTP.

```bash
docker run --rm -p 8080:80 -v $PWD:/usr/local/apache2/htdocs httpd
```

Going back to your browser and refreshing the page, you will now find the `from-docker` folder and the `info.txt` within
that. You are able to open this file in your browser, now.

If you want your container to run as daemon in background, you can start your container with the `-d` option:

```bash
docker run --rm -p 8080:80 -v $PWD:/usr/local/apache2/htdocs -d --name server httpd
```

It will return the ID of your created container. With this ID you can apply other command of the Docker CLI on them.
When you add the `--name` attribute you can identify the container also with that, in a much more human readable way.
You can list all running containers with the following command:

```bash
docker ps
```

### Run commands on running containers

You can also run commands on running containers. It works similar to the `docker run` command:

````bash
docker exec server ps aux
````

Also interactive with shell:

````bash
docker exec -it server bash --norc
````

Let's check the anatomy of this command:

1. `docker`: is the application on our operating system, of course.
2. `exec`: is our command to execute a command on a running container.
3. `-it`: A parameter. You can add as much parameters as you want, but only at this place, so not before or after this
step. In this case we use it in an interactive way with a shell.
4. `server`: This is the name or ID of the container where we want to run the command.
5. `bash`: This is the command we want to run in our container. This part is mandatory on the exec command, it will not
use the default command of the image here!
6. `--norc`: Parameters for the command in step five. This value is optional.

In the point of view of virtualization this command is similar to a SSH login to your virtual machine.

### Stop, restart, remove a service

To stop a service we can use the `docker stop [container-name-or-id]` command followed by the ID or name of the
container. After that we can start a service again with the `docker start [container-name-or-id]` command, but just as
long as it was not created with the `--rm` option of `docker run`. To remove and delete a service use the
`docker rm [container-name-or-id]` command.

### Compose your containers

For orchestrating multiple containers of your micro-services into one service `docker-compose` is the perfact match. For
using `docker-compose` on Linux you have to download it. On Windows and MacOS it is already included in the
bundle.

Let's orchestrate our HTTP server with `docker-compose`. At first you have to create a file called `docker-compose.yml`
in your base directory. The file should have the following content to create a server like created before with
standalone docker.

```yaml
version: "3"
services:
  server:
    image: httpd
    ports:
      - 8080:80
    volumes:
      - ./:/usr/local/apache2/htdocs
```

*Note: relative paths are allowed here, but have to be prefixed with `./`*

Now you can start your server with the following command:

````bash
docker-compose up -d
````

The `docker-compose` CLI is verbose as the normal `docker` CLI. You can just run a command prefixed with `--help` and
get additional information. To get a list of possible commands you can simply run
`docker-compose`. The structure of this CLI is very similar to the original `docker` CLI, but it is optimized for
orchestration. You can spawn multiple micro-services at once with that configuration file called `docker-compose.yml`.

## Create a SDI with Docker

This is the practical part of this workshop. To prevent problems and keep it easy we created a git repository with a
branch for every step.

You can clone the repository with this command:

```bash
git clone https://github.com/yagajs/workshop-docker-geo-basics-lesson.git
```

### MapProxy

MapProxy is a very easy to install application. The YAGA Development Team has written a Dockerfile and has published a
image on Docker-Hub already for you. This makes it very easy to start and run it. Use the following `docker-compose.yml`
file:

```yaml
version: "3"
services:
  osm-mapproxy:
    image: yagajs/mapproxy
    expose:
      - 8080
    ports:
      - "127.0.0.1:8080:8080"
```

Start it with `docker-compose up` and watch the result on [`http://localhost:8080`](http://localhost:8080).

#### Configure OSM as OWS service

At the moment we just use a demo-server. Let's add the original OSM server as source by saving the following content to
`mapproxy/osm/mapproxy.yaml`:

*The configuration is based on:
[`https://github.com/mapproxy/mapproxy/blob/master/mapproxy/config_template/base_config/mapproxy.yaml`](https://github.com/mapproxy/mapproxy/blob/master/mapproxy/config_template/base_config/mapproxy.yaml)*

```yaml
services:
  demo:
  tms:
    use_grid_names: true
    # origin for /tiles service
    origin: 'nw'
  kml:
      use_grid_names: true
  wmts:
  wms:
    md:
      title: MapProxy WMS Proxy for OSM
      abstract: This is a minimal MapProxy example.

layers:
  - name: osm
    title: OpenStreetMap
    sources: [osm_cache]

caches:
  osm_cache:
    grids: [webmercator]
    sources: [osm_source]

sources:
  osm_source:
    type: tile
    grid: osm_grid
    url: http://tile.openstreetmap.org/%(z)s/%(x)s/%(y)s.png

grids:
    webmercator:
      base: GLOBAL_WEBMERCATOR
    osm_grid:
      base: GLOBAL_MERCATOR
      srs: 'EPSG:3857'
      origin: nw

globals:
```

Now we have to insert this configuration in our MapProxy container, by enhancing the `docker-compose.yml`:

```yaml
version: "3"
services:
  osm-mapproxy:
    image: yagajs/mapproxy
    expose:
      - 8080
    ports:
      - "127.0.0.1:8080:8080"
    volumes:
      - ./mapproxy/osm:/docker-entrypoint-initmapproxy.d
```

Now restart your container by terminating the running container with `Ctrl-C` and staring again with `docker-compose up`
again. After that, ry to open [`http://localhost:8080`](http://localhost:8080) again and use the original OSM map.

### GeoServer

In our next exercise we will add GeoServer to our dockerized SDI.

```yaml
version: "3"
services:
  # Your other services...
  geoserver:
    image: yagajs/geoserver
    expose:
      - 8080
    ports:
      - "127.0.0.1:8081:8080"
```

After restarting the docker containers again, like described in the step before, you can access GeoServer on
[`http://localhost:8081/geoserver`](http://localhost:8081/geoserver) and login with the default username (`admin`) and
password (`geoserver`).

### PostGIS

PostGIS is a spatial extension to PostgreSQL. It is currently the relational database with the best performance and
support for spatial queries. We want to use it to store spatial vector data and want to connect it to our GeoServer.


Add this part to your `docker-compose.yml`:
```yaml
version: "3"
services:
  # Your other services...
  postgis:
    image: yagajs/postgis
    expose:
      - 5432
    environment:
      POSTGRES_PASSWORD: "my-secret-one"
      POSTGRES_USER: "yaga"
      POSTGRES_DB: "yaga"
    volumes:
      - postgis-data:/var/lib/postgresql/data
    ports:
      - "127.0.0.1:5432:5432"
volumes:
  postgis-data:
```

#### Activating PostGIS extension

To activate the PostGIS extension we need to add a provisioning script that enables it. Save it as this file:
`postgis/provision.sh`.

```bash
#!/usr/bin/env bash

if [ `psql --username postgres -d $POSTGRES_DB -c "SELECT extname from pg_extension;" | grep postgis | wc -l` != 0 ] ;then
  echo "Provisioning already done..."
  exit
fi

echo "Enable Postgis support"
psql --username postgres -d $POSTGRES_DB -c "CREATE EXTENSION postgis;"

```

The file should be an executable (`chmod +x postgis/provision.sh`).

In addition we have to mount this script to the `/docker-entrypoint-initdb.d` by extending the `docker-compose.yml` with

```yaml
services:
  postgis:
    volumes:
      - postgis-data:/var/lib/postgresql/data
      - ./postgis:/docker-entrypoint-initdb.d
```

*This is just a striped version of the complete `docker-compose.yml`.*

Restart your containers again.

### Connect GeoServer with PostGIS

Login into your created GeoServer with the default username and password (`admin`:`geoserver`) like described above.

Add a datastore on the web-application. Click on the green add button, to add a new datastore. Select PostGIS.

Fill out the following inputs as follows:

* host: `postgis` (this corresponds to the name of the service in our `docker-compose.yml`)
* port: `5432`
* database: `yaga`
* schema: `public`
* user: `yaga`
* password: `my-secret-one`

*If you like, add layers to this database. You are also able to connect to the PostGIS database from your local QGIS.
You have to use `localhost:5432` from your host system!* 

## Additional notes

This is just a beginner guide! There is a lot of stuff you can do with Docker and there are a lot of further best
practices that will not fit into a simple beginners guide.

The YAGA Development is a group of OpenSource enthusiasts with a connection to the geo-environment. We build already a
bunch of images for Docker. If you think there is an important image missing you can contact us at
[info@yagajs.org](mailto:info@yagajs.org). Please keep in mind that we can't give you a full-service support and quick
responses on that basis as "volunteers". We try to answer issues within a week. If you want to get professional support
you can write an e-mail to [info@spatialists.com](mailto:info@spatialists.com).

If you have an offline version of this workshop, take a look at the Git Repository
[`https://github.com/yagajs/workshop-docker-geo-basics-booklet`](https://github.com/yagajs/workshop-docker-geo-basics-booklet)
for updates.

## Security

You should not bind a port of a container to your host system as long as you really need it. When you just need it for
administrative purpose you should access your container from the CLI, or open a port just for your localhost!

## Scaling

Prefer using a Reverse-Proxy like NGINX as gateway for your requests. You can scale NGINX easily. If you want to provide
multiple cached-layers: create one MapProxy for each layer. Keep in mind that you are building **micro**-services with
Docker! You should use this concept for all the other images, too. Scaling of micro-services is only reasonable possible
if you have metrics, so collect them! Docker has a build-in ability to scale over multiple physical instances with
Docker Swarm.

## Appendix: List of useful docker commands

* `docker run -it [image-name] [optional command]`: Run docker in interactive mode.
* `docker run --rm [image-name] [optional command]`: Delete docker container after the command has finished.
* `docker stop $(docker ps -q)`: Stop all running containers.
* `docker exec -it [name-of-container] /bin/bash`: Run bash on a running container.
* `docker exec -it [name-of-container] /bin/sh`: Run sh on a running container (use this for Alpine images!).
* `docker inspect [name-of-container]`: Get detailed information about your container.

**practical, but dangerous commands**

*These commands are practical especially to tidy-up your personal computer. You should execute these commands on a server
very carefully!*

* `docker rm $(docker ps -aq)`: Remove all stopped containers.
* `docker volume rm $(docker volume ls -q)`: Remove all unmounted volumes.
* `docker rmi $(docker image ls -q)`: Remove all unused images.
* `docker system prune`: Remove all images, containers, volumes and networks.
