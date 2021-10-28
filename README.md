# Docker Compose Stack Deployment Guide

1. Clone this repository
2. Run `cd freeradius-admin`
3. Run `docker-compose up --build -d`
4. Run `docker exec -it fradmin-nginx chown -R www-data:www-data /var/www/html`
5. Run `docker-compose down`
6. Run 'docker-compose up -d'
7. Using your browser go to [http://localhost:80](http://localhost:80). Enter the login credentials detailed in the SOP.
8. To test the RADIUS server run `docker container exec -it fradmin-radtest /bin/sh`. 
9. Run `radtest username password 127.0.0.1:1812 0 testing123`.

# Important Information

1. The folder _./mysql/src/data_ must be emptied to prevent any conflicts with database passwords
2. Edit [docker-compose.yml](docker-compose.yml). Change the `MYSQL_ROOT_PASSWORD` entry to your own password.
3. Edit [./web/src/.env](./web/src/.env) by changing the database settings to match the above.
4. Edit [./freeradius/config/freeradius/mods-enabled/sql](sql) with the correct database root password.

# Securing Each Deployment 

1. Each time this stack is deployed in production it is HIGHLY recommended that you generate a new application key.
2. To do this, Navigate to the _./web/src_ directory and run the following command `php artisan key:generate'.
3. Rebuild the images by running `docker-compose build` at the project root.
4. Run `docker-compose up -d` to run the services in the background.
5. Confirm the services are up by running `docker container ps`.

# Backend Configuration 

**mysql**

The **fradmin-mysql** container is configured with a generic username and password. This should be changed in the [docker-compose.yml](docker-compose.yml) file for production setups. The file [radius.sql](./mysql/srv/initdb.d/radius.sql) is a modified version of the radius MySQL schema included with [freeRADIUS](https://github.com/FreeRADIUS/freeradius-server) distributions. It is loaded by the container on startup if the database and tables do not already exist. Furthermore, it is set to port-forward 3306 in case you need direct network access to the MySQL service. If this sort of access is not needed, it should be disabled in production setups by changing the 'port' declaration to 'expose'.

**freeradius 3.0.17**

The FreeRADIUS 3 server is the service around which the rest of the services in this app are built. You will need some knowledge about configuring it in order to make it work according to your requirements. The configuration files are located in the folder _./freeradius/config/freeradius_. This folder is volume-mapped to As is, these files are ready to be deployed and will instantiate one virtual server called **server01** listening on the default auth and acct ports 1812 and 1813. Depending on your use case, you may wish to edit the _Simultaneous-Use_ section of the [queries.conf](./freeradius/src/mods-config/sql/main/mysql/queries.conf) file.
