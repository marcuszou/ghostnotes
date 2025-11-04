# AlfaNotes
by Marcus Zou



## Table of Contents

- [Intro](#intro)
- [The Enterprise / Production Edition: mysql / mariadb based](#The Enterprise / Production Edition: mysql / mariadb based)
- [Running up the docker](#Running up the docker)
- [Trouble-shooting](#Trouble-shooting)
- [The Primary Edition: sqlite3-based](#The Primary Edition: sqlite3-based)
- [Enjoy the show](#Enjoy the show)
- [Pullover the webapp](#Pullover the webapp)
- [Outro](#Outro)
- [License](#License)



## Intro

This is the Alfa-Notes WebApp utilizing the best open-sourced blogging platform of https://ghost.org. 

Let's roll up sleeves and get hands dirty.

1- Create folders

```shell
## create project folder
mkdir alfanotes && cd alfanotes
## make sub-folders
mkdir -p assets db content
```

2- Select database: A simple way is to use the file based sqlite3 for small loading projects; but production-ready database shall be MySQL 8.x or MariaDB 11.x, even a PostgreSQL instance.



## The Enterprise / Production Edition: mysql based

#### Feature

1. Based on **mysql 8.4** or **mariadb 11.4**, great for enterprise/big project
2. Stable and popular database engine 



#### The `yaml` file

The contents of the `docker-compose.yaml` shall be like below:

```textfile
services:
  ghost:
    image: ghost:6-alpine
    container_name: anotes
    restart: always
    ports:
      - 2368:2368
    depends_on:
      - dbsvr
    environment:
      database__client: mysql
      database__connection__host: dbsvr
      database__connection__user: zenusr
      database__connection__password: strongPass!2024
      database__connection__database: anotes
      url: http://localhost:2368
      # NODE_ENV: production
    volumes:
      - ./content:/var/lib/ghost/content

  dbsvr:
    image: mysql:8.4
    container_name: anotes-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: superStrong!2024
      MYSQL_USER: zenusr
      MYSQL_PASSWORD: strongPass!2024
      MYSQL_DATABASE: anotes
    volumes:
      - ./dbmysql:/var/lib/mysql

volumes:
  content:
  db:
```

Quite a lot of time that many geeks get stuck at a unstable docker of the main container: __anotes__. Please check out the tips below:

> [!TIP]
>
> Please ensure running up the containers with these values:
>
> - `database__client` must be set to `mysql` whatever your database is mysql 8.x or mariadb 8.x - 11.x, even 14.x.
> - `database__connection__host` should reference to the **service name** (not the container) of your database.
> - `MYSQL_DATABASE` and `database__connection__database` need to be the same values.
> - `MYSQL_USER` and `database__connection__user` need to be the same values.
> - `MYSQL_PASSWORD` and `database__connection__password` need to be the same values.
>
> Otherwise you will suffer from an issue of `ERROR Invalid database host`.

> [!TIP]
>
> The **mysql** is more picky than **mariadb**, then please ensure this is no file in __db__ folder prior to mounting and starting.



## Running up the docker

```shell
docker compose up -d  
## If running up a sepecific ymal file, please do
## docker compose -f docker-compose-prod-mariadb.yaml up -d
```

Open a browser with a link: http://localhost:2368 for the website. you shall receive a output like:

![brb](./assets/webapp_brb.png)

> [!IMPORTANT]
>
> It takes __3~5 minutes__ (It is a very long time) to initialize the __mariadb__ database and configure the default Ghost Blog engine, then be patient.

> [!CAUTION]
>
> It takes __24 minutes__ to finish off the entire process using __mysql 8.0__ database, on my Dell Precision 7820 Tower monster (48 CPU cores, 256GB RAM, SATA HDD though), then be PATIENT!

Once you see a message of "__Explore Response 200 OK__", the webapp screen shall be like:

![normal](./assets/webapp_normal.png)

You are good to go for the `setup` page: http://localhost:2368/ghost, which will bring up a screen of:

![setup](./assets/webapp_setup.png)



## Trouble-shooting

There might be problematic when running up the containers.

1- A very good command knowing if the main container, __anotes__, is up running or not can be conducted as below:

```shell
docker logs anotes
```

At first, the logs may indicate the main container is shutting down, etc. but wait for a moment and try this command again. 

2- Sometime, the __db__ folder does not have any files mounted, while the __content__ folder got mounted very well. then the issue is definitely related to the database server. then, run the command below to diagnose the database server:

```shell
docker logs anotes-db
```

3- The more often issue of the database container, __anotes-db__, is the container is broken. then launch a command:

```shell
docker exec -it anotes-db bash
```

Then run a command to login to the mysql/mariadb instance:

```shell
mysql -u zenusr -p
## You have to type in the strongPass!2024 as password 
```

Once connected, you can check out if a database instance is in place or not:

```shell
SHOW DATABASES;
```

Press "exit" to quit the session.



## Trouble-shooting mysql 8.x

While `mariadb` is so great, the `mysql 8.x` could introduces a lot of issues:

#### `mysql` 8.0

1. The typical issue is the docker has no writing privilege to bind-mount against a local folder; then please remove such mounting (the `VOLUMES` section inside the database container), but using `docker logs <db-container>` command to check up. This will avoid 95% of the issue, pretty sure!
2. The initialization takes around **20-30 minutes**, then be patient.

#### `mysql` 8.4

1. The typical issue is the docker has no writing privilege to bind-mount against a local folder or `mysql_ssl_rsa_setup: Can't change permissions of the file 'ca-key.pem'`, that's due to the Linux environment inside the Docker container (and WSL2's Linux VM) may have difficulty interpreting or modifying permissions on files originating from a Windows NTFS filesystem. This can lead to the "**Operation not permitted**" error when `mysqld` tries to set specific permissions for its sensitive files like `ca.pem`

2. Solution: 

   Remove such mounting (the `VOLUMES` section inside the database container). This will avoid 95% of the issue, pretty sure! 

   The initialization takes around **4 minutes**, that's way faster than `mysql8.0`, then be patient.

3. Check up: Run the command below to check out what's going on:

   ```shell
   docker logs anotes-db
   ```

   you should have the following output:

   > [!IMPORTANT]
   >
   > 2025-11-04 06:46:39+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.4.7-1.el9 started.
   > 2025-11-04 06:46:45+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
   > 2025-11-04 06:46:46+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.4.7-1.el9 started.
   > 2025-11-04 06:46:47+00:00 [Note] [Entrypoint]: Initializing database files
   > 2025-11-04T06:46:47.094451Z 0 [System] [MY-015017] [Server] MySQL Server Initialization - start.
   > 2025-11-04T06:46:47.120032Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.4.7) initializing of server in progress as process 81
   > 2025-11-04T06:47:54.162314Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
   > 2025-11-04 06:49:05+00:00 [Note] [Entrypoint]: Database files initialized
   > 2025-11-04 06:49:05+00:00 [Note] [Entrypoint]: Starting temporary server
   > 2025-11-04T06:49:06.792891Z 0 [System] [MY-015015] [Server] MySQL Server - start.
   > 2025-11-04T06:49:07.296579Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.4.7) starting as process 145
   > 2025-11-04T06:49:07.795843Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
   > 2025-11-04T06:49:23.120455Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
   > 2025-11-04T06:49:29.386350Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
   > 2025-11-04T06:49:29.386562Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
   > 2025-11-04T06:49:29.530947Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
   > 2025-11-04T06:49:29.821967Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: /var/run/mysqld/mysqlx.sock
   > 2025-11-04T06:49:29.822779Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.4.7'  socket: '/var/run/mysqld/mysqld.sock'  port: 0  MySQL Community Server - GPL.
   > 2025-11-04 06:49:29+00:00 [Note] [Entrypoint]: Temporary server started.
   > '/var/lib/mysql/mysql.sock' -> '/var/run/mysqld/mysqld.sock'
   > Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
   > Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
   > Warning: Unable to load '/usr/share/zoneinfo/leapseconds' as time zone. Skipping it.
   > Warning: Unable to load '/usr/share/zoneinfo/tzdata.zi' as time zone. Skipping it.
   > Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
   > Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.
   > 2025-11-04 06:49:56+00:00 [Note] [Entrypoint]: Creating database anotes
   > 2025-11-04 06:49:57+00:00 [Note] [Entrypoint]: Creating user zenusr
   > 2025-11-04 06:49:57+00:00 [Note] [Entrypoint]: Giving user zenusr access to schema anotes
   >
   > 2025-11-04 06:49:57+00:00 [Note] [Entrypoint]: Stopping temporary server
   > 2025-11-04T06:49:57.945168Z 13 [System] [MY-013172] [Server] Received SHUTDOWN from user root. Shutting down mysqld (Version: 8.4.7).
   > 2025-11-04T06:50:04.358608Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.4.7)  MySQL Community Server - GPL.
   > 2025-11-04T06:50:04.358684Z 0 [System] [MY-015016] [Server] MySQL Server - end.
   > 2025-11-04 06:50:04+00:00 [Note] [Entrypoint]: Temporary server stopped
   >
   > 2025-11-04 06:50:04+00:00 [Note] [Entrypoint]: MySQL init process done. Ready for start up.
   >
   > 2025-11-04T06:50:05.021952Z 0 [System] [MY-015015] [Server] MySQL Server - start.
   > 2025-11-04T06:50:05.403980Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.4.7) starting as process 1
   > 2025-11-04T06:50:05.482067Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
   > 2025-11-04T06:50:21.889740Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
   > 2025-11-04T06:50:28.564089Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
   > 2025-11-04T06:50:28.564250Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
   > 2025-11-04T06:50:28.625015Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
   > 2025-11-04T06:50:28.803688Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
   > 2025-11-04T06:50:28.804676Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.4.7'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.

   

4. In case that you persist bind-mounting the `/var/lib/mysql` folder from the container to the host, 

5. Solution:

   1. Mount a docker volume, instead of a folder/volume on the host. Basically change

      ```shell
      volumes:
        - ./db:/var/lib/mysql
      ```

      to 

      ```shell
      volumes:
        - db:/var/lib/mysql
      ```

      As you know, the first setting is to mount the `/var/lib/mysql` from the container to a folder `db` on the host. and the second case represents to mount the same `/var/lib/mysql` from the container to an internal folder `db` on the container. that makes huge difference.

   2. If you are using WSL2 Linux, please change the WSL2 Metadata by entering the following into `/etc/wsl.conf` within the WSL2 instance:

      ```shell
      [automount]
      options = "metadata,umask=22,fmask=11"
      ```

      After modifying, run `wsl --shutdown` and restart your WSL2 instance.



## The Primary Edition: sqlite3-based

#### Features

1. Light-weight, using the `sqlite3` database (a single file) 

2. Fast and great fit for small-sized development project and test-out


#### The `yaml` file
The `docker-compose.yml` can be referred as below

```textfile
services:
  ghost:
    image: ghost:6-alpine
    container_name: anotes
    restart: always
    ports:
      - 2368:2368
    environment:
      database__client: sqlite3
      database__connection__filename: /var/lib/ghost/content/data/anotes.db
      url: http://localhost:2368
      # NODE_ENV: development
    volumes:
      - ./content:/var/lib/ghost/content/
      - ./dbsqlite:/var/lib/ghost/content/data/

volumes:
  content:
  dbsqlite:
```

#### Running up the docker

```shell
docker-compose up -d  ## primary edition: sqlite3 database
```





## Enjoy the show

Finally open a browser, and type: http://localhost:2368 for the website.
First time? Please go to: http://localhost:2368/ghost for the admin page.



## Pullover the webapp

Using `docker compose down` command:

```shell
docker compose down --remove-orphans
```



As simple as below:

```shell
# Take down the 2 containers
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
docker ps  ## there is no such 2 containers any more

# Remove the images
docker image rm $(docker images -q)
docker images  ## there is no such 2 images any more
```



## Outro

You may go to https://ghost.org/ for more details.



## License

MIT
