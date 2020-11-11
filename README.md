# owncloud-server
Raspberry Pi arm64 owncloud server

## End Result
These instructions, and the associated docker-compose.yml file will result in a Raspbian arm64 system running the latest arm64 owncloud community server, using the latest arm64 versions of both MariaDB and Redis.

## High level steps for Raspbian arm64 setup
1. Install Raspbian 64-bit
1. Install the latest version of Docker
1. Install pyenv/Python dependencies
1. Install docker-compose

For detailed instructions see the following, [Raspbian arm64 setup](https://github.com/tooratty/owncloud-server)

## High level steps for Owncloud server setup
These steps need to be performed on an account that has Docker access. By default this is the root user.

1. Clone this repository into a local dicretory
1. Create a .env file with the required KVPs
1. Start Owncloud with docker-compose
1. Monitor logs for any issues

### Cloning this repository
[Cloning A Repository](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/cloning-a-repository)

### .env file setup
The following KVPs are required,
Key | Used For
--- | --------
DB_ROOT_PASSWORD | MariaDB root password
DB_NAME          | Name of the Owncloud DB
DB_USER          | User with R/W access to the Owncloud DB
DB_PASSWORD      | Password for the the aforementioned user
ADMIN_USERNAME   | Owncloud administrator username
ADMIN_PASSWORD   | Owncloud administrator password

Here's an example of a .env file,
```
DB_ROOT_PASSWORD=myrootpassword
DB_NAME=owncloud
DB_USER=owncloud
DB_PASSWORD=owncloud
ADMIN_USERNAME=admin
ADMIN_PASSWORD=myadminpassword
```

### Starting the Owncloud server
The first startup is critical for getting MariaDB and Owncloud setup properly. If the first startup fails for any reason it's probably easier to fix the .env file or docker-compose.yml file, and completely clean out the Docker names volumes and re-try the first startup.

```
docker-compose up -d
```

#### Starting from scratch
**NOTE**, this operation will irrevocably destroy the MariaDB, Redis and Onwcloud volumes. Do not perform this operation if you're not clear about what it should be used for.

```
docker-compose down
docker volume rm owncloud-server_backup
docker volume rm owncloud-server_files
docker volume rm owncloud-server_mysql
docker volume rm owncloud-server_redis
docker system prune
```

### Monitoring the MariaDB, Redis and Owncloud logs
To view the MariaDB logs use,
```
docker-compose logs db --follow
```

To view the Redis logs use,
```
docker-compose logs redis --follow
```

To view the Owncloud logs use,
```
docker-compose logs owncloud --follow
```

## Other helpful docker-compose commands
To stop everything use,
```
docker-compose stop
```

To restart everything use,
```
docker-compose restart
```

## Upgrading Mariadb or Redis to the latest security patches
The default docker-compose.yml file calls out version 10.5 for MariaDB and version 6.0 for Redis. As security updates become available, the following set of docker-compose commands will upgrade both MariaDB and Redis to the latest releases of their respective major/minor versions.

```
docker-compose pull
docker-compose down
docker-compose up -d
```

## Upgrading Owncloud
Upgrading to a new version of Owncloud is a bit more involved. It requires putting Owncloud into maintenance mode, and from my expereice manually moving it back to running mode after the Docker container is up and running.

```
docker-compose exec owncloud occ maintenance:mode --on
docker-compose exec db backup
```

Now, update the Owncloud version in the docker-compse.yml file.

```
docker-compose pull
docker-compose down
docker-compose up -d
```

Finally, monitor the logs and if Owncloud doesn't automatically come out of maintenance mode run the following,

```
docker-compose exec owncloud occ maintenance:mode --off
```

Official documentation is available here, [Upgrading Owncloud on Docker](https://doc.owncloud.com/server/admin_manual/installation/docker/#upgrading-owncloud-on-docker).
