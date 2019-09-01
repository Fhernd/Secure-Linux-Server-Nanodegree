

# Project 2 - Linux Server Configuration

## Description
Takes a baseline installation of a Linux server and prepare it to host your web applications. Secure a Linux server from a number of attack vectors, install and configure a database server, and deploy one an existing web application (*Item Catalog*) onto it.

## Basic configuration

Create a virtual server on Amazon Lightsail. This server was equiped with Ubuntu 16.04 LTS.

The administrator login with PuTTY with the given user and key file provided in the Amazone Lightsail console.

Make an update and upgrade

	sudo apt-get update
	sudo apt-get upgrade

## Change SSH default port
Execute this command from the terminal:

	sudo vi /etc/ssh/sshd_config

Change port from 22 to **2200** and save the file.

Then restart the `sshd` service with:

	sudo service sshd restart

**Note**: If you are using a panel to admin the VPS (virtual server), eventually you need to open de port directly from that panel to allow incoming connections to the virtual server.

## Firewall configuration

Open these ports with `ufw` command:

	sudo ufw deny 22
	sudo ufw allow www
	sudo ufw allow 2200/tcp
	sudo ufw enable

## Create `grader` user

1. `sudo adduser grader`
2. `sudo touch /etc/sudoers.d/grader`
3. Add  this content to the file:
`grader ALL=(ALL:ALL) ALL`
4. Save the file.

## Generate Key Pairs
From a local machine using a terminal like Git-Bash, generate a key pairs:

	ssh-keygen

Follow the required steps: (1) the destination folder, and (2) the passphrase

## Installing the public key

The public key generated must be installed in the virtual server. Follow these steps from  the terminal:
1. `su - grader`
2. `mkdir .ssh`
3. `touch .ssh/authorized_keys`
4. Add the public key content generated in the previous section and save the file.

Additional permissions must be done:

5. `chmod 700 .ssh`
6. `chmod 644 .ssh/authorized_keys`

Finally, reload the SSH service:

7. `service ssh restart`

## Test SSH connection from the SSH client

From the Git Bash (or similar) establish an SSH connection:

	ssh grader@54.158.156.25 -p 2200 -i «path_to_private_key»

## PostgreSQL configuration & Installation

1. `sudo apt-get install postgresql`  
2. `sudo su - postgres`
3. `psql`
4. Database and user creation  
    ```
    postgres=# CREATE DATABASE catalog;
    postgres=# CREATE USER catalog;
    ```
6. Password for the new user:
    ```
    postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
    ```
7. Permissions for the new user:
    ```
    postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
    ```
8. `postgres=# \q`
9.  `exit`

## Project deployment

1. Git installation is required to clone the project at [CatalogItem-Nanodegree-Udacity](https://github.com/Fhernd/CatalogItem-Nanodegree-Udacity)

		sudo apt-get install git
2. `cd /var/www`
3. `mkdir FlaskApp`
4. `cd FlaskApp`
5. Clone the mentioned project

		sudo git clone https://github.com/Fhernd/CatalogItem-Nanodegree-Udacity
6. `sudo mv CatalogItem-Nanodegree-Udacity FlaskApp`
7. `cd FlaskApp`
8. `mv project.py __init__.py`
9. If required, install:

		sudo apt-get install libpq-dev
		sudo pip3 install --upgrade pyasn1-modules
		
10. `sudo pip3 install psycopg2`
11. `sudo python database_setup.py`
12. `sudo python populate.py`

## Apache Web Server Installation & Configuration

**Note**: These steps was perfomed by the default user the virtual server has provided.

### Installation

1. `sudo apt-get install apache2`
2. `sudo apt-get install libapache2-mod-wsgi-py3`
3. `sudo service apache2 restart`


### Configure a virtual host

1. Create a virtual host

		sudo vi /etc/apache2/sites-available/FlaskApp.conf

2. Add the follow content to the file

		<VirtualHost *:80>
			ServerName ec2-54-158-156-25.compute-1.amazonaws.com
			ServerAdmin webmaster@server.com
			WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
			<Directory /var/www/FlaskApp/FlaskApp/>
				Order allow,deny
				Allow from all
			</Directory>
			Alias /static /var/www/FlaskApp/FlaskApp/static
			<Directory /var/www/FlaskApp/FlaskApp/static/>
				Order allow,deny
				Allow from all
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			LogLevel warn
			CustomLog ${APACHE_LOG_DIR}/access.log combined
		</VirtualHost>

**Note**: The folders in the paths for the virtual host will be created in the next section.

3. Disable the current virtual host:

		sudo a2dissite 000-default.conf

4. Enbable the new virtual host:

		sudo a2ensite FlaskApp

## WSGI entry point

Now, it's required to create a WSGI configuration file to start the Python Flask web application on Apache Web server:

1. `cd /var/www/FlaskApp`
2. `sudo vi flaskapp.wsgi`

File content for `flaskapp.wsgi`:

	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application

## Dependencies

This project depends on the following **additional** Python modules installation:

* `$ sudo pip3 install sqlalchemy`
* `$ sudo pip3 install flask`
* `$ sudo pip3 install google-auth`
* `$ sudo pip3 install flask-cors`

## Execution

[http://54.158.156.25.xip.io](http://54.158.156.25.xip.io)

## Workaround

With an IP address there is no way to add a JavaScript origin to the Google Console, so searching on the Internet I found that [xip.io/](http://xip.io/) was the solution to this exception. Defintion from official website: *xip.io is a magic domain name that provides wildcard DNS for any IP address.*

## Support material

* [How To Configure Apache Virtual Hosts In Ubuntu 18.04 LTS - OSTechNix](https://www.ostechnix.com/configure-apache-virtual-hosts-ubuntu-part-1/)
* [Google: Permission denied to generate login hint for target domain NOT on localhost - Stack Overflow](https://stackoverflow.com/questions/36020374/google-permission-denied-to-generate-login-hint-for-target-domain-not-on-localh)
* [amazon ec2 - Update Ubuntu 10.04 - Server Fault](https://serverfault.com/questions/262751/update-ubuntu-10-04/262773#262773)
* Guide for configuration: [https://github.com/jungleBadger/-nanodegree-linux-server](https://github.com/jungleBadger/-nanodegree-linux-server)
