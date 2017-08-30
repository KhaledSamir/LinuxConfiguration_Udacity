# Linux-Server-Configuration

In this project , I'll create an instance ( Server ) on by using Amazon Lightsail to run Ubuntu and deployed on of python applications that I developed through out the course.

My Catalog Application can be visited here :  http://18.220.148.146/

## Instructions for SSH access to the instance
- Create a file and type in the Private key and name it KhaledUbuntu.
- Move the private key file into the folder `~/.ssh`.
- Open your terminal and type in
	```chmod 600 ~/.ssh/KhaledUbuntu```
- In your terminal, type in
	```ssh -i ~/.ssh/KhaledUbuntu root@18.220.148.146```


## How to create a new user " grader " and grant Sudoer access
- `sudo adduser grader` and put the information needed.
- `vim /etc/sudoers`
- `touch /etc/sudoers.d/grader`
- `vim /etc/sudoers.d/grader`, type in
 `grader ALL=(ALL:ALL) ALL` then save and quit vim App.


	## On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ vim .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys

	```
So, we can access these new keys.

 We should be able to test the connection between your local machine and the remote server by running :

	`ssh -i ~./ssh/KhaledUbuntu grader@18.220.148.146`

## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## How to change the SSH port from 22 to 2200
1. Use `sudo vim /etc/ssh/sshd_config` and at line 4 ,change Port 22 to Port 2200 , save.
2. Type in `sudo service ssh restart` to restart the ssh, now in Amazon Lightsail , we can't use " Connect using SSH " button anymore because we changed the port and it wouldn't be able to connect using 22 anymore.

## Configure the Uncomplicated Firewall (UFW)

	sudo ufw allow www
	sudo ufw allow ntp
  sudo ufw allow 2200/tcp
	sudo ufw enable

## Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2- There will be a page to select different option , choose other and then UTC.

## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
- Install PostgreSQL `sudo apt-get install postgresql`
- Login as user "postgres" `sudo su - postgres`
- Get into postgreSQL shell `psql`
- Create a new database named catalog  and create a new user named catalog in postgreSQL shell

	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
- Set a password for user catalog

	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'khaled1234';
	```
- Give user "catalog" permission to "catalog" application database

	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
- Quit postgreSQL `^C [Control + C]`
- Exit from user "postgres" , type in

	```
  exit
	```

## Install git and setup the Catalog Application: ( Need to switch to Amazon branch after clone )

- Install Git using `sudo apt-get install git`
- Use `cd /var/www` to move to the /var/www directory
- Create the application directory `sudo mkdir FlaskApp`
- Move inside this directory using `cd FlaskApp`
- Clone the Catalog App to the virtual machine `git clone https://github.com/KhaledSamir/Catalog-Item.git`
- You need to run " sudo git checkout Amazon " to switch to Amazon branch ( Ready for deployment )
- Rename the project's name `sudo mv ./Catalog-Item ./FlaskApp`
- Move to the inner FlaskApp directory using `cd FlaskApp`
- Install psycopg2 `sudo apt-get install postgresql python-psycopg2`
- Install pip `sudo apt-get install python-pip`
- Install dependencies by typing in `sudo pip install -r requirements.txt`
- Create database schema `sudo python database_setup.py`

## Configure the Virtual Env
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host.

	```
  <VirtualHost *:80>
        ServerName 18.220.148.146
        ServerAdmin dev.khaledsaleh@gmail.com
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
	```
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Create and configure the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp:

	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi
	```
2. Add the following lines of code to the flaskapp.wsgi file:

	```
  #!/usr/bin/python
 import sys
 import logging
 logging.basicConfig(stream=sys.stderr)
 sys.path.insert(0,"/var/www/FlaskApp/")

 from FlaskApp import app as application
 application.secret_key = 'Add your secret key'
	```

## Restart Apache
Restart Apache `sudo service apache2 restart `

## References:
I followed this link to be able to deploy to Amazon Lightsail instance :

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
