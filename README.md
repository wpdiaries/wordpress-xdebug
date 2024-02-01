WordPress with XDebug for Docker
================================

**It is intended for development environment only. Please do not use this in production environment. Please you the [official Docker WordPress image](https://hub.docker.com/_/wordpress) at production instead.**

XDebug has been added to the official WordPress Docker image here. The tag of the image corresponds to the tag of the official WordPress image.

For example, `wpdiaries/wordpress-xdebug:6.4.2-php8.2-apache` means that the image has been created based on the image `wordpress:6.4.2-php8.2-apache`.

But if you do not want to use the precompiled image, but wish to build the image yourself, you can clone this Git repository with the command:

```
git clone https://github.com/wpdiaries/wordpress-xdebug.git xdebug
```

And then you could use a `docker-compose.yml` similar to this one (of course you would need to substitute your own parameters where necessary):

```yaml
version: '3.8'

services:

  wordpress:
    container_name: wordpress-wpd
    restart: always
    build:
      dockerfile: Dockerfile # this line is actually redundant here - you need it only if you want to use some custom name for your Dockerfile
      context: ./xdebug # a path to a directory containing a Dockerfile, or a url to a git repository

    ports:
      - "80:80"

    environment:
      VIRTUAL_HOST: mydomain.com, www.mydomain.com
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: mydbname
      WORDPRESS_DB_USER: mydbuser
      WORDPRESS_DB_PASSWORD: mydbpassword
      # Set the XDEBUG_CONFIG as described here: https://xdebug.org/docs/remote
      XDEBUG_CONFIG: client_host=host.docker.internal

    depends_on:
      - db

    volumes:
      - /opt/projects/wpd/www:/var/www/html

    networks:
      - backend-wpd
      - frontend-wpd


  db:
    container_name: mysql-wpd
    image: mysql:8.0.33
    command: --default-authentication-plugin=mysql_native_password
    restart: always

    environment:
      MYSQL_ROOT_PASSWORD: mydbrootpassword
      #MYSQL_RANDOM_ROOT_PASSWORD: '1' # You can use this instead of the option right above if you do not want to be able login to MySQL under root
      MYSQL_DATABASE: mydbname
      MYSQL_USER: mydbuser
      MYSQL_PASSWORD: mydbpassword

    ports:
      -  "3306:3306" # I prefer to keep the ports available for external connections in the Development environment to be able to work with the database
                     # from programs like e.g. HeidiSQL on Windows or DBeaver on Mac.

    volumes:
      - /opt/projects/wpd/mysql:/var/lib/mysql

    networks:
      - backend-wpd


networks:
  frontend-wpd:
  backend-wpd:
```

Please notice that we have added the following environment variable to your `docker-compose.yml`:
```
XDEBUG_CONFIG: remote_host=host.docker.internal
```
, DNS name `host.docker.internal` automatically resolves to the internal IP address used by the host, if want to update replace `host.docker.internal` with the IP address of your host-machine (where you have your IDE, e.g. `PhpStorm` installed).

The variable `XDEBUG_CONFIG` is an XDebug environment variable. It allows you to add or redefine some XDebug configuration parameters. More information could be found [here](https://xdebug.org/docs/remote).

Of course you should never use such configuration in the Production environment. It is for the Development environment only.

E.g. we have port 3306 here open for external connections. So we will be able to connect to MySQL from programs like e.g. HeidiSQL on Windows or DBeaver on Mac. This is convenient for Development. But I doubt anyone would do anything like that on the Production server.

You can find more examples of using `wpdiaries/wordpress-xdebug` [in this article on wpdiaries.com](https://www.wpdiaries.com/wordpress-with-xdebug-for-docker/).

  
