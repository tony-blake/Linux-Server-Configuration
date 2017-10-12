LinuxServerConfiguration
===========================================

Description
-----------

**LinuxServerConfiguration** outlines the steps involved in configuring an online Linux server to host a web application. It is the final project in the Full Stack Web Developer Nanodegree

## Information for grader

* contents of private RSA file is in ```LightsailDefaultPrivateKey.pem```
* IP address of server - 52.208.13.234
* Username for grader - "grader"
* Server login password for grader - "Passw0rd1"
* Pasword for sudo access - "Passw0rd"
* Port - 2200
* URL - http://ec2-54-165-131-195.compute-1.amazonaws.com

## Steps to login for grader
* Open up a terminal window and type ```ssh -v grader@52.208.13.234 -p 2220```
* When prompted  for the password for the ssh key ```id_rsa``` type ```Passw0rd1```  
* The password when using ```sudo``` is ```Passw0rd```



## Obtain access to virtual server

Set up amazon lightsail account using steps from "Get started on Lightsail" in project prep.
https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175462/lessons/3573679011239847/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e. 

Download the default private key from the Account page.
Then move private key to .ssh folder.
Log into server.

```bash
mv ~/Downloads/LightsailDefaultPrivateKey.pem ~/.ssh/
chmod 600 ~/.ssh/LightsailDefaultPrivateKey.pem 
ssh -i ~/.ssh/LightsailDefaultPrivateKey.pem ubuntu@52.208.13.234

```


## Update and upgrade all packages

To update all packages use

```bash
sudo apt-get update
sudo apt-get upgrade
```

## Change the SSH port from 22 to 2200 and reconfigure authetications
```bash
sudo vim /etc/ssh/sshd_config
```
Then change ```Port 22``` to ```Port 2200```

## Conigure Amazon Lightsail Firewall

Click on "Networking" tab. 

Under Firewall section click "add another" .

Choose "Custom" as application, "TCP" as protocol and "2200" as range

## Configure Uncomplicated Firewall (UFW)

Check UFW status to make sure its inactive ```sudo ufw status```

Deny all incoming by default ```sudo ufw default deny incoming```

Allow outgoing by default ```sudo ufw default allow outgoing```

Allow SSH on port 2200 ```sudo ufw allow 2200/tcp```

Allow HTTP on port 80 ``` sudo ufw allow 80/tcp```

Allow NTP on port 123 ```sudo ufw allow 123/udp ```

Turn on firewall ```sudo ufw enable```




## Create a new user called grader

```bash
sudo adduser grader

```
When prompted enter the following user details for grader

```bash
Enter new UNIX password: Passw0rd
Retype new UNIX password: Passw0rd
passwd: password updated successfully
Changing the user information for grader
Enter the new value, or press ENTER for the default
	Full Name []: Gradey.McGraderson
	Room Number []: 80085
	Work Phone []: 420
	Home Phone []: 42000
	Other []: 206-709-3100
Is the information correct? [Y/n] Y

```

Change the ssh-config file so that grader has the permission to use sudo.

```bash
sudo visudo
```
The file opnes in nano and you can insert the line ```grader ALL=(ALL:ALL) ALL ``` 

under  ``` "#User privilege specification" ``` underneath ```root ALL=(ALL:ALL) ALL ``` . 

Then save file by hitting ``` ctrl + x ``` and selecting Yes.


## Create ssh login keys for grader

On local machine generate SSH key pair by running command ``` ssh-keygen ```

```bash
Last login: Thu Oct 12 10:42:47 on ttys001
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/tonyblake/.ssh/id_rsa): /Users/tonyblake/.ssh/id_rsa12Oct17
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/tonyblake/.ssh/id_rsa12Oct17.
Your public key has been saved in /Users/tonyblake/.ssh/id_rsa12Oct17.pub.
The key fingerprint is:
ae:ea:95:98:c8:d7:0d:d9:25:79:8b:ad:58:ab:86:09 tonyblake@Macintosh-109add6f31eb.local
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|         .       |
|        o o      |
|       o * .     |
|      o S o      |
| .E. + B o       |
|  o.+o= =        |
|   .o..o         |
|   .ooo          |
+-----------------+


Macintosh-109add6f31eb:~ tonyblake$ 

```

Then in the virtual machine swich superuser to grader ``` su - grader ``` where password is ```Passw0rd```

Create .ssh directory ```mkdir .ssh```

Create empty file to store key ``` vim .ssh/authorized_keys ```

On local machine read contents of the public key ``` cat ~/.ssh/rsa_id.pub```

Copy the key and paste into the authorized_keys file in grader ```vim .ssh/authorized_keys```

Set permissions for files: ```chmod 700 .ssh```  and ```chmod 644 .ssh/authorized_keys```
 
run ```sudo service ssh restart``` for changes to take effect

logout of grader

logout of ubuntu

The terminal output should look like so
 
 ```bash

ubuntu@ip-172-26-14-122:~$ su - grader
Password: 
grader@ip-172-26-14-122:~$ vim .ssh/authorized_keys
grader@ip-172-26-14-122:~$ chmod 700 .ssh
grader@ip-172-26-14-122:~$ chmod 644 .ssh/authorized_keys
grader@ip-172-26-14-122:~$ sudo service ssh restart
[sudo] password for grader: 
grader@ip-172-26-14-122:~$ 
```






## Configure the local timezone to UTC
 
run ```sudo dpkg-reconfigure tzdata```

Then when prompted select "none of the above". 

Then select "UTC".

## Install and configure Apache to serve a Python mod_wsgi application

run ``` sudo apt-get install apache2``` 

Type public ip address (54.165.131.195) into URL and check if apache webpage "it works!" is there 

install mod_wsgi: ```sudo apt-get install libapache2-mod-wsgi```


## Install and Configure PostgreSQL

To obtain and install postgresql just run ```sudo apt-get install postgresql postgresql-contrib```

Check config file ```/etc/postgresql/9.5/main/pg_hba.conf``` only allows connections from host addresses ``` 127.0.0.1``` for IPv4 and ```::1``` for IPv6. It should look like so

```bash 
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            md5
#host    replication     postgres        ::1/128                 md5
```

Next create a postgresql user called ``` catalog ``` with ```sudo -u postgres createuser -P catalog```

When the prompt for a password appaears type ```Passw0rd```

Then create an empty database called ```catalog``` using ```sudo -u postgres createdb -O catalog catalog```

## Install Flask and all other dependencies

create new file called ```requirements``` like so

```bash 
# create new file
sudo vim requirements

# install dependencies 
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask-seasurf
```
Then run ```bash requirements```

## Install Git and clone catalog app repository 

```bash
sudo apt-get install git
sudo mkdir fullstack-nanodegree-vm
sudo chown www-data:www-data fullstack-nanodegree-vm/
sudo -u www-data git clone https://github.com/tony-blake/TrueSophia.git fullstack-nanodegree-vm
```

The www-data user will be used to run the catalog app.

## Update the OAuth client secrets file

First find host name from IP address

```bash
Macintosh-109add6f31eb:~ tonyblake$ host 54.165.131.195
195.131.165.54.in-addr.arpa domain name pointer ec2-54-165-131-195.compute-1.amazonaws.com
```

To get the Google+ authorization working:
Go to the Developer Console: https://console.developers.google.com/ 

Click on Credentials in the sidebar and then click edit icon on the far right of the client id

add your host name and public IP-address to your Authorized JavaScript origins 

and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-54-165-131-195.compute-1.amazonaws.com/oauth2callback

Download new json and paste contents into new file ```g_client_secrets.json``` 

To get the Facebook authorization working:
Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes"

## Organise folder hierarchy and updated files to match paths in virtual host config file

```bash

cd fullstack-nanodegree-vm/
ls
cd ..
cp -r fullstack-nanodegree-vm/Desktop/Sophia/* fullstack-nanodegree-vm/
sudo cp -r fullstack-nanodegree-vm/Desktop/Sophia/* fullstack-nanodegree-vm/
cd fullstack-nanodegree-vm/
ls
rm -fr Desktop/
sudo rm -fr Desktop/
sudo rm README.md
cd ..
ls
cd fullstack-nanodegree-vm/
ls
cd vagrant/
ls
cd catalog/
ls
sudo vim catalog.wsgi
cd ..
cp g_client_secrets.json /var/www/catalog/fullstack-nanodegree-vm/vagrant/catalog/
sudo cp g_client_secrets.json /var/www/catalog/fullstack-nanodegree-vm/vagrant/catalog/
sudo cp fb_client_secrets.json /var/www/catalog/fullstack-nanodegree-vm/vagrant/catalog/
cd /var/www/catalog/
rm -fr TrueSophia/
cd /var/www/
ls
sudo mkdir fullstack-nanodegree-vm
cd catalog/
ls
cp requirements /var/www/catalog/fullstack-nanodegree-vm/vagrant/catalog/
sudo cp requirements /var/www/catalog/fullstack-nanodegree-vm/vagrant/catalog/
cd fullstack-nanodegree-vm/
ls
cd ..
ls
sudo chown www-data:www-data fullstack-nanodegree-vm/
sudo cp -r catalog/fullstack-nanodegree-vm/* fullstack-nanodegree-vm/
sudo rm -fr catalog/
ls
cd fullstack-nanodegree-vm/
ls
cd vagrant/
ls
cd catalog/
ls
sudo vim catalog.wsgi
ls
sudo vim application.py
sudo vim /etc/apache2/sites-available/catalog-app.conf
cd /etc/apache2/sites-available/
ls
sudo a2dissite 000-default.conf
sudo a2ensite catalog-app.conf
sudo service apache reload
sudo service apache2 reload
sudo service apache2 restart
```

## Update catalog.wsgi

To get the application to run the catalog.wsgi file needs to know the location of the catalog app files. So change contents of the old catalog.wsgi to the follwing

```bash
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/fullstack-nanodegree-vm/vagrant/catalog')

from catalog import app as application
from catalog.database_setup import create_db
from catalog.populate_database import populate_database

application.secret_key = 'super_secret_key'  # This needs changing in production env

application.config['DATABASE_URL'] = 'postgresql://catalog:PASSWORD@localhost/catalog'
application.config['UPLOAD_FOLDER'] = '/var/www/fullstack-nanodegree-vm/vagrant/catalog/item_images'
application.config['OAUTH_SECRETS_LOCATION'] = '/var/www/fullstack-nanodegree-vm/vagrant/catalog/'
application.config['ALLOWED_EXTENSIONS'] = set(['pdf','jpg', 'jpeg', 'png', 'gif'])
application.config['MAX_CONTENT_LENGTH'] = 200 * 1024 * 1024  # 4 MB

# Create database and populate it, if not already done so.
create_db(application.config['DATABASE_URL'])
populate_database()
```

## Configure Apache2 to run catalog application

A virtual host configuration file needs to be created in the directory ```/etc/apache2/sites-available/```
called ```catalog-app.conf```

And it should have the follwoing contents 

```bash
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        ServerName 54.165.131.195

        ServerAlias ec2-54-165-131-195.compute-1.amazonaws.com


        # Define WSGI parameters. The daemon process runs as the www-data user.
        WSGIDaemonProcess catalog user=www-data group=www-data threads=5
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}

        # Define the location of the app's WSGI file
        WSGIScriptAlias / /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog.wsgi

        # Allow Apache to serve the WSGI app from the catalog app directory
        <Directory /var/www/fullstack-nanodegree-vm/vagrant/catalog/>
                Require all granted
        </Directory>

        # Setup the static directory (contains CSS, Javascript, etc.)
        Alias /static /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/static

        # Allow Apache to serve the files from the static directory
        <Directory  /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/static/>
                Require all granted
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Disable the default virtual host with ```sudo a2dissite 000-default.conf```

Then enable the catalog app virtual host ```sudo a2ensite catalog-app.conf```


## Finally run application!

restart apache server ```sudo service apache reload```

load webpage by pasting URL ```ec2-54-165-131-195.compute-1.amazonaws.com``` into web browser 

## Addressing "internal server error" complaint if webpage does not load

All errors are placed in the log file which can be viewed (last 20 lines of file) using 

```sudo tail -20 /var/log/apache2/error.log```








 





















