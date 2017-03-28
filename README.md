LinuxServerConfiguration
===========================================

Description
-----------

**LinuxServerConfiguration** outlines the steps involved in configuring an online Linux server to host a web application. It os the final project in the Full Stack Web Developer Nanodegree

## Information for Grader

* IP address - 54.165.131.195
* Port - 2200
* URL -


## Obtain access to virtual server

Set up amazon lightsail account using steps from "Get started on Lightsail" in project prep.
https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175462/lessons/3573679011239847/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e. 

Download the default private key from the Account page.
Then move private key to .ssh folder.
Log into server.

```bash
mv ~/Downloads/LightsailDefaultPrivateKey.pem ~/.ssh/
chmod 600 ~/.ssh/LightsailDefaultPrivateKey.pem 
ssh -i ~/.ssh/LightsailDefaultPrivateKey.pem ubuntu@54.165.131.195

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
Last login: Thu Mar 23 23:04:51 on ttys000
Macintosh-109add6f31eb:~ tonyblake$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/tonyblake/.ssh/id_rsa): /Users/tonyblake/.ssh/id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/tonyblake/.ssh/id_rsa.
Your public key has been saved in /Users/tonyblake/.ssh/id_rsa.pub.
The key fingerprint is:
6a:da:e8:78:ff:39:75:60:70:5f:4f:98:db:00:dd:f2 tonyblake@Macintosh-109add6f31eb.local
The key's randomart image is:
+--[ RSA 2048]----+
|            .o + |
|        . .   B o|
|         o . . O |
|          o . . E|
|        S. .     |
|       .  . .    |
|      o  . .     |
|   ..=  ..       |
|  .o+.o.o.       |
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

Type public ip address (54.165.131.195) into URL and check if apache webpage is there (sceenshot)

install mod_wsgi: ```sudo apt-get install libapache2-mod-wsgi```

Next use the WSGI module to configure the apche server to handle requests ```sudo vim /etc/apache2/sites-enabled/000-default.conf```

Add ```WSGIScriptAlias / /var/www/html/myapp.wsgi``` before ```</ VirtualHost>```

Restart Apache ```sudo apache2ctl restart```

## Restart Error (If it happens)

The following error message appears on attempting to restart
```bash
AH00558: apache2: Could not reliably determine the servers fully qualified domain name, using 127.0.0.1. 
Set the ServerName directive globally to suppress this message.
```

To resolve this ``` sudo vim  /etc/apache2/apache2.conf```
And append the following line ```ServerName localhost```

## Application Installation

Install Git
```bash
sudo apt-get install git
git config --global user.name "GRADEY"
git config --global user.email "GRADEY@54.165.131.195.com"

```


## Install and configure PostgreSQL

run ```sudo apt-get install postgresql postgresql-contrib```

Check if no remote connections are allowed ```sudo vim /etc/postgresql/9.5/main/pg_hba.conf```
```bash
# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```

Create a PostgreSQL user called catalog ```sudo -u postgres createuser -P catalog```

When prompted to create password type ```Passw0rd2```

Create an empty database called catalog ```sudo -u postgres createdb -O catalog catalog```

## Install Applications

run the following commands

```bash
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask-seasurf
```

## Clone the repository that contains Catalog app

run the following commands

```bash
cd /srv/
sudo mkdir fullstack-nanodegree-vm
sudo chown www-data:www-data fullstack-nanodegree-vm/
sudo -u www-data git clone https://github.com/tony-blake/TrueSophia.git fullstack-nanodegree-vm
```


## Update catalog.wsgi file

The following changes have been made. 
* Paths changed to where catalog is located.
* Secret key application set to a random value
* PostgreSQL for ```catalog``` user is set

```bash
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/srv/fullstack-nanodegree-vm/Desktop/Sophia/vagrant/catalog')

from catalog import app as application
from catalog.database_setup import create_db
from catalog.populate_database import populate_database

application.secret_key = 'SECRET'  # This needs changing in production env

application.config['DATABASE_URL'] = 'postgresql://catalog:PASSWORD@localhost/catalog'
application.config['UPLOAD_FOLDER'] = '/srv/fullstack-nanodegree-vm/Desktop/Sophia/vagrant/catalog/item_images'
application.config['OAUTH_SECRETS_LOCATION'] = '/srv/fullstack-nanodegree-vm/Desktop/Sophia/vagrant/catalog'
application.config['ALLOWED_EXTENSIONS'] = set(['jpg', 'jpeg', 'png', 'gif'])
application.config['MAX_CONTENT_LENGTH'] = 200 * 1024 * 1024  # 200 MB

# Create database and populate it, if not already done so.
create_db(application.config['DATABASE_URL'])
populate_database()
```


## Configure Virtual Apache 2 Server to host  Catalog App



Install python dev and verify WSGI is enabled

```sudo apt-get install python-dev```
   sudo a2enmod wsgi
   



```
## Required Libraries and Dependencies
The project code requires the following software:

* Python 2.7.x
* [SQLAlchemy](http://www.sqlalchemy.org/) 0.8.4 or higher (a Python SQL toolkit)
* [Flask](http://flask.pocoo.org/) 0.10.1 or higher (a web development microframework)
* The following Python packages:
    * oauth2client
    * requests
    * httplib2
    * flask-seasurf (a CSRF defence)


You can run the project in a Vagrant managed virtual machine (VM) which includes all the
required dependencies (see below for how to run the VM). For this you will need
[Vagrant](https://www.vagrantup.com/downloads) and
[VirtualBox](https://www.virtualbox.org/wiki/Downloads) software installed on your
system.

## Project contents
This project consists for the following files in the `catalog` directory:

* `application.py` - The main Python script that serves the website. If no database
    is found, one is created and populated by `populate_database.py`.
* `fb_client_secrets.json` - Client secrets for Facebook OAuth login.
* `g_client_secrets.json` - Client secrets for Google OAuth login.
* `README.md` - This read me file.
* `/catalog` - Directory containing the `catalog` package.
    * `/static` - Directory containing CSS and Javascript for the website.
        Includes a copy of the [Material Design Lite](http://www.getmdl.io/)
        web framework by Google and
        [JavaScript Cookie](https://github.com/js-cookie/js-cookie/), a JavaScript
        API for handling cookies.
    * `/templates` - Directory containing the HTML templates for the website, using
        the [Jinja 2](http://jinja.pocoo.org/docs/dev/) templating language for Python.
        See next section for more details on contents.
    * `__init__.py` - Initialises the Flask app and imports the URL routes.
    * `auth.py` - Handles the login and logout of users using OAuth.
    * `connect_to_database.py` - Function for connecting to the database.
    * `database_setup.py` - Defines the database classes and creates an empty database.
    * `file_management.py` - Functions for handling user uploaded image files.
    * `json_endpoint.py` - Returns the data in JSON format.
    * `populate_database.py` - Inserts a selection of animals into the database.
    * `views.py` - Provides backend code to produce web page views of the data and forms
        for creating, editing and deleting animals. It also ensures that only the user
        that added an animal can edit or delete it.
    * `xml_generator.py` - Returns the data in XML format.

### Templates
The `/templates` directory contains the following files, written in HTML and the Jinja2
templating language:

* `add_item_button.html` - Renders a MDL button on a page to add a new item (paper).
* `delete_item.html` - Delete paper confirmation page.
* `edit_item.html` - Form to edit the details of a paper.
* `homepage.html` - The default page, which lists the 10 latest papers that were added.
* `item.html` - A page that displays a single paper and its details.
* `items.html` - A page that lists the papers belonging to a single category.
* `layout.html` - This defines the common layout of the website and is the parent
    for all the other template pages.
* `login.html` - A login page featuring OAuth Goolge+ and Facebook login buttons.
* `my_items.html` - A page for displaying the pages you have added to the website.
* `new_item.html` - A form for adding a new paper.

## How to Run the Project
Download the project zip file to you computer and unzip the file. Or clone this
repository to your desktop.

Open the text-based interface for your operating system (e.g. the terminal
window in Linux, the command prompt in Windows).

Navigate to the project directory and then enter the `vagrant` directory.

### Bringing the VM up
Bring up the VM with the following command:

```bash
vagrant up
```

The first time you run this command it will take awhile, as the VM image needs to
be downloaded.

You can then log into the VM with the following command:

```bash
vagrant ssh
```

More detailed instructions for installing the Vagrant VM can be found
[here](https://www.udacity.com/wiki/ud197/install-vagrant).

### Make sure you're in the right place
Once inside the VM, navigate to the tournament directory with this command:

```bash
cd /vagrant/catalog
```

### OAuth setup
In order to log in to the web app, you will need to get either a Google+ or Facebook
(or both) OAuth app ID and secret. For Google, go to the
[Google Developers Console](https://console.developers.google.com/) and for Facebook,
go to [Facebook Login](https://developers.facebook.com/products/login).

Once you have your credentials, put the IDs and secrets in the `fb_client_secrets.json`
file for Facebook and `g_client_secrets.json` for Google.

You will now be able to log in to the app.

### Run application.py
On the first run of `application.py` there will be no database present, so it creates
one and populates it with sample data. On the command line do:

```bash
python application.py
```

It then starts a web server that serves the application. To view the application,
go to the following address using a browser on the host system:

```
http://localhost:8000/
```

You should see the latest papers that were added to the database. Have a read of the papers in the different categories. You can download them by right clicking on the pdf image in the item category. To upload a document from your own device , you'll need to log in first with either a Google or Facebook account.


### Shutting the VM down
When you are finished with the VM, press `Ctrl-D` to logout of it and shut it down
with this command:

```bash
vagrant halt
```

## Extra Credit Description
The following features are present to earn an extra credit from Udacity.

### XML Endpoint
Database contents can be obtained in XML form by going to
[http://localhost:8000/catalog.xml](http://localhost:8000/catalog.xml). Depending
on your browser, you may need to view source of the page to view the XML file. You
can do this by right-clicking and selecting `View Page Source` from the menu.

### Papers
The Sophia app allows the user to add, edit and delete an image for each paper. An
image may either be uploaded to the app or specified with the URL of an image. If
a paper has an image associated with it, it is displayed to all users.

### Cross-site Request Forgery Protection
The app uses the Flask plugin [SeaSurf](https://flask-seasurf.readthedocs.org/en/latest/),
in order to protect against cross-site request forgery. An addition cookie is stored
in the client containing a nonce. When a user logs in, adds, edits or deletes an item,
the contents of this cookie is checked to make sure it matches the value on the server.
If it does not match or is missing, no change to the database will be made.

### References
I modeled the layout of my "Journal App" on the "Zoo Management App" by Steven Wooding.
https://github.com/SteveWooding/fullstack-nanodegree-vm/tree/master/vagrant/catalog 



