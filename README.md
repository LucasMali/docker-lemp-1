## docker-lemp


> Do not use this LEMP in Production.
> For production, use [adhocore/phpfpm](https://github.com/adhocore/docker-phpfpm)
> then [compose](https://docs.docker.com/compose/install/) a stack using individual `nginx`, `redis`, `mysql` etc images.



It is quick jumpstart for onboarding you into docker based development.
The download size is just about ~360MB which is tiny considering how much tools and stuffs it contains.


> `*`: Actually [MariaDB 10.6.9](https://mariadb.com/kb/en/mariadb-vs-mysql-compatibility/).

> `~`: RC version can be used for test/dev but not production.

> `+`: Different image tags each viz `8.2`, `:8.1`, `:8.0` and `:7.4`.

## Usage

Install [docker](https://docs.docker.com/install/) in your machine.
Also recommended to install [docker-compose](https://docs.docker.com/compose/install/).

```sh
# pull latest image
docker build -t lemp .

# If you want to setup MySQL credentials, pass env vars
docker run -p 80:80 -p 88:88 -v /Users/lucas/Development/docker-lemp/public:/var/www/html/public \
  -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=wp \
  -e MYSQL_USER=user -e MYSQL_PASSWORD=password \
  --name lemp -d lemp
  # for postgres you can pass in similar env as for mysql but with PGSQL_ prefix
```

After running container as above, you will be able to browse [localhost](http://localhost)!

The database adminer will be available for [mysql](http://localhost:80/adminer?server=127.0.0.1%3A3306&username=root)
and [postgres](http://localhost:80/adminer?pgsql=127.0.0.1%3A5432&username=postgres).

The mailcatcher will be available at [localhost:8888](http://localhost:8888) which displays mails in realtime.

### Stop container

To stop the container, you would run:

```sh
docker stop lemp
```

### (Re)Start container

You dont have to always do `docker run` as in above unless you removed or lost your `lemp` container.

Instead, you can just start when needed:

```sh
docker start lemp
```

> **PRO** If you develop multiple apps, you can create multiple lemp containers with different names.
>
> eg: `docker run -p 8081:80 -v $(pwd):/var/www/html --name new-lemp -d adhocore/lemp:8.0`


## With Docker compose

Create a `docker-compose.yml` in your project root with contents something similar to:

```yaml
# ./docker-compose.yml
version: '3'

services:
  app:
    image: adhocore/lemp:8.0
    # For different app you can use different names. (eg: )
    container_name: some-app
    volumes:
      # app source code
      - ./path/to/your/app:/var/www/html
      # db data persistence
      - db_data:/var/lib/mysql
      # Here you can also volume php ini settings
      # - /path/to/zz-overrides:/usr/local/etc/php/conf.d/zz-overrides.ini
    ports:
      - 80:80
    environment:
      MYSQL_ROOT_PASSWORD: supersecurepwd
      MYSQL_DATABASE: appdb
      MYSQL_USER: dbusr
      MYSQL_PASSWORD: securepwd
      # for postgres you can pass in similar env as for mysql but with PGSQL_ prefix

volumes:
  db_data: {}
```

Then all you gotta do is:

```sh
# To start
docker-compose up -d

# To stop
docker-compose stop
```

As you can see using compose is very neat, intuitive and easy.
Plus you can already set the volumes and ports there, so you dont have to type in terminal.

### MySQL Default credentials

- **root password**: 1234567890 (if `MYSQL_ROOT_PASSWORD` is not passed)
- **user password**: 123456 (if `MYSQL_USER` is passed but `MYSQL_PASSWORD` is not)

### PgSQL Default credentials

- **postgres password**: 1234567890 (if `PGSQL_ROOT_PASSWORD` is not passed)
- **user password**: 123456 (if `PGSQL_USER` is passed but `PGSQL_PASSWORD` is not)


#### Accessing DB

In PHP app you can access MySQL db via PDO like so:
```php
$db = new PDO(
    'mysql:host=127.0.0.1;port=3306;dbname=' . getenv('MYSQL_DATABASE'),
    getenv('MYSQL_USER'),
    getenv('MYSQL_PASSWORD')
);
```

You can access PgSQL db via PDO like so:
```php
$pdb = new PDO(
    'pgsql:host=127.0.0.1;port=5432;dbname=' . getenv('PGSQL_DATABASE'),
    getenv('PGSQL_USER'),
    getenv('PGSQL_PASSWORD')
);
```

### Nginx

URL rewrite is already enabled for you.

Either your app has `public/` folder or not, the rewrite adapts automatically.

### PHP

For available extensions, check [adhocore/phpfpm#extensions](https://github.com/adhocore/docker-phpfpm#extensions).

### Disabling services

[Pass in env var](https://www.cloudsavvyit.com/14081/how-to-pass-environment-variables-to-docker-containers/)
`DISABLE` to the container in CSV format to disable services.
The service names must be one or more of below in comma separated format:
```
beanstalkd
mailcatcher
memcached
mysql
pgsql
redis
```

> Example: `DISABLE=beanstalkd,mailcatcher,memcached,pgsql,redis`
> Essential services like `nginx`, `php`, `adminer` cannot be disabled ;).

The service(s) will be enabled again if you run the container next time without `DISABLE` env or if you remove specific services from `DISABLE` CSV.

### Testing mailcatcher

```sh
# open shell
docker exec -it lemp sh

# send test mail
echo "\n" | sendmail -S 0 test@localhost
```

Then you will see the new mail in realtime at http://localhost:8888.

Or you can check it in shell as well:
```sh

curl 0:88/messages
```
