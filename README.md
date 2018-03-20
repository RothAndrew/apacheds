Apache DS in a Docker Container
===============================

Apache DS is a Java implementation of Directory server (**LDAP**). This projects puts it into container and makes it easier to configure and bootstrap it with some data.

[Apache DS Page](1) that explains what it is.

## Docker Compose
`docker-compose` makes it very easy to build and deploy Apache DS in one step.

Doing this will:

1. Build the image
1. Expose the ports necessary to connect to Apache DS
1. Create the volumes necessary to persist your data and bootstrap the app
1. Specifies a file called `apacheds_admin_password.secret` that you can put the password for your system admin account.

See the [docker-compose.yml](docker-compose.yml) file for full details.

To deploy Apache DS, run the following commands:

    git clone https://github.com/rothandrew/apacheds.git
	cd apacheds
	docker-compose up -d --build

The default instance of Apache DS will now be running on port 10389 with no SSL. It has a default admin user and password and all default Apache DS schemas.

> user: **uid=admin,ou=system** password: **secret**

## Build and run the container without Docker Compose

	git clone https://github.com/rothandrew/apacheds.git
	cd apacheds/build
	docker build -t apacheds:latest .
	docker run -d -p 10389:10389 apacheds:latest

This starts a default instance of Apache DS running on port 10389 with no SSL. It has a default admin user and password and all default Apache DS schemas. 

> bind dn: **uid=admin,ou=system** password: **secret**

Note: If you do it this way, without using Docker Compose, you can't change the system admin password. If you do, the next time you restart the container it won't be able to start because the bootstrapping script relies on a [Docker Secret](https://docs.docker.com/engine/swarm/secrets/) to provide the password and Secrets aren't available to standalone containers.

## Changing the System Admin password

The docker compose file uses a Docker Secret linked to a file called `apacheds_admin_password.secret`. `apacheds_admin_password.secret` is git-ignored so it won't be there when you clone a fresh instance. To change the system admin password to, follow the following steps:

1. Start Apache DS using `docker-compose up -d --build`
1. Log in using [Apache Directory Studio](http://directory.apache.org/studio/) using the above default credentials
1. Change the password according to the [documentation](http://directory.apache.org/apacheds/basic-ug/1.4.2-changing-admin-password.html)
1. `docker-compose stop`
1. `echo "MyNewP@ssw0rd" >> apacheds_admin_password.secret`
1. `docker-compose start`

Note: Docker may have inadvertently created a **directory** called `apacheds_admin_password.secret` when you ran `docker-compose up`. Make sure you delete it before creating the actual secret file.

## Configuration 
Container has two volumes defined:

* **/data** - if you want to persist container data somewhere. [docker-compose.yml](docker-compose.yml) creates a [Docker Volume](https://docs.docker.com/storage/volumes/) called `apacheds_data`
* **/bootstrap** - for configuration, schema and bootstraping file

### Bootstrapping

If you want to bootstrap Apache DS with a specific **schema** place Apache DS specific schema in the [./bootstrap](bootstrap) folder, then start your server using `docker-compose`. The docker-compose file has that folder as a volume to be used for bootstrapping.

Sample schema can be found in a [sample/schema.tar](sample/schema.tar) in this GitHub repository. 

If you want to use specific **config.ldif** to setup Apache DS place it in the [./bootstrap](bootstrap) directory. Docker Compose will pick it up automaticaly.

If you have some extra data you want to put into Directory, you can place a file in the [./bootstrap](bootstrap) folder and setup Environment variable **BOOTSTRAP_FILE** in [docker-compose.yml](docker-compose.yml):

	services:
      ldap:
        environment:
          - BOOTSTRAP_FILE=/bootstrap/the-file.ldif

## Change ports

To change which port you want to use to connect to Apache DS, modify [docker-compose.yml](docker-compose.yml). For example, if you want to connect using port 20389 rather than 10389:

    services:
      ldap:
        ports:
          - "20389:10389"

## Using SSL

Please check back later for instructions on how to secure Apache DS using SSL.

## Lifecycle Management

To create & start:

    docker-compose up -d --build

> `-d` Tells docker-compose to run in detached mode. `--build` tells docker-compose to build the docker image first in case there have been any changes.

To stop:

    docker-compose stop

To start:

    docker-compose start

To restart:

    docker-compose restart

To completely tear down the instance and delete all data:

    docker-compose down -v

> If you don't use the -v flag, the containers will be stopped and removed, but the persistent volume will not.


### Examples

[sample](sample) folder containes example schema, example config file and example of a data creation in LDAP upon bootstrap.

> Pull request for contributions are always WELCOME! :)