# Mark's Modifications to this Project

This fork of the [dodeeric/omeka-s-docker](https://github.com/dodeeric/omeka-s-docker) project introduces a new `docker-compose.yml` file for spinning this up on any Dockerized server, and a Docksal `.docksal` directory to enable local development using `fin up`.

System requirements for development of this project currently include:

- [Docker (Community Edition)](https://docs.docker.com/install/)
- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Docksal](https://docksal.io)

The workflows mentioned here were created on a Mac workstation and successfully pushed to a staging environment running Ubuntu 16.04.

## Local Development and Testing

If your workstation is able to run the aforementioned required components then the following steps can be used to launch and develop a local instance.  In a terminal on your workstation with Docker running...

```
cd ~/Projects    # or any path of your choice
git clone https://github.com/McFateM/omeka-s-docker.git
cd omeka-s-docker
fin up
```

The `fin up` command in this sequence should launch the project locally.  It should report...
```
Project URL: http://omeka-s-docker.docksal:8080
```
However, that statement is not entirely accurate in the case of Omeka-S.  The correct address will NOT require the `:8080` suffix.  So, the target address should be:
```
http://omeka-s-docker.docksal:8080
```

You should be able to start (`fin up`) and stop (`fin down`) this local project as often as needed.  Any data you add to Omeka in this mode should persist as long as you don't remove the Omeka container or reset Docker entirely.

## Deploy to Staging

The requirements for an Omeka-S staging environment are essentially the same as the local workstation, except that Docksal is NOT required.

In addition, the staging server needs to also have been configured using `./init` and the workflow outlined in my [docker-bootstrap Workflow](https://static.grinnell.edu/blogs/McFateM/posts/008-docker-bootstrap-workflow/).

If all of those requirements have been met the site can be deployed to staging (currently this involves the `dgdocker2.grinnell.edu` server). In a terminal open to DGDocker2 with Docker running...

```
cd ~/Projects    # or any path of your choice
git clone https://github.com/McFateM/omeka-s-docker.git
cd omeka-s-docker
docker-compose up -d
```

You should be able to start (`docker-compose up -d`) and stop (`docker-compose stop`) this project as often as needed.  Any data you add to Omeka in this mode should persist as long as you don't remove the Omeka container or reset Docker entirely.

| Note |
| --- |
| What follows is from the original [dodeeric/omeka-s-docker](https://github.com/dodeeric/omeka-s-docker) `README.md` file. |


# Omeka-S in Docker Containers

## Launch the containers

Install Docker and Docker-compose on your host (can be a physical or virtual machine).

Download the file "docker-compose.yml".

From the directory containing the "docker-compose.yml" file:

```
$ sudo docker-compose up -d
```

This will deploy three Docker containers:

- Container 1: mariadb (mysql)
- Container 2: phpmyadmin (connected to container 1)
- Container 3: omeka-s (connected to container 1)

With your browser, go to:

- Omeka-S: http://hostname
- PhpMyAdmin: http://hostname:8080

At that point, you can start configuring your Omeka-S web portal.

Remarks:

- images will be downloaded automatically from the Docker hub: mariadb:latest, phpmyadmin:latest, dodeeric/omeka-s:latest.
- for the omeka-s container, /var/www/html/files (media files uploaded by the users) and /var/www/html/config/database.ini (configuration file with the credentials for the db) are put in a named volume and will survive the removal of the container. The mariadb container also puts the data (omeka-s db in /var/lib/mysql) in a named volume. Volumes are hosted in the host filesystem (/var/lib/docker/volumes).

To stop the containers:

```
$ sudo docker-compose stop
```

To remove the containers:

```
$ sudo docker-compose rm
```

Remark: this will NOT delete the volumes (omeka and mariadb). If you launch again "sudo docker-compose up -d", the volumes will be re-used.

To login into a container:

```
$ sudo docker container exec -it <container-id-or-name> bash
```

## Build a new image (optional)

If you want to modify the omeka-s image (by changing the Dockerfile file), you will need to build a new image:

E.g.:

```
$ git clone https://github.com/dodeeric/omeka-s-docker.git
$ cd omeka-s-docker
```

Edit the Dockerfile file.

Once done, build the new Docker image:

```
$ sudo docker image build -t foo/omeka-s:1.0.1-bar .
$ sudo docker image tag foo/omeka-s:1.0.1-bar foo/omeka-s:latest
```

Upload the image to your Docker hub repository:

Login in your account (e.g. foo) on hub.docker.com, and create a repository "omeka-s", then upload your customized image:

```
$ sudo docker login --username=foo
$ sudo docker image push foo/omeka-s:1.0.1-bar
$ sudo docker image push foo/omeka-s:latest
```

## Use Traefik as proxy (optional)

If you want to access all your web services on port 80 (or 443), you can use the Traefik reverse proxy and load balancer.

Here we have 3 web servers running (phpmyadmin, omeka-s, gramps). All are reachable on port 80 after launching this command:

```
$ sudo docker-compose -f docker-compose-traefik.yml up -d
```

All xxx.dodeeric.be dns names are directed to the Traefik container which will proxy them to the corresponding service container. The xxx.dodeeric.be dns names have to point to the IP of the Docker host.

With your browser, go to: (dodeeric.be is replaced by your dns domain; e.g. mydomain.com)

- Omeka-S: http://omeka.mydomain.com
- PhpMyAdmin: http://pma.mydomain.com
- Gramps: http://gramps.mydomain.com

Traefik has a management web interface: http://hostname:8080

Only the Traefik container exposes its TCP ports (80, 443, 8080) on the Docker host; the service containers run on the private "network1" network.
