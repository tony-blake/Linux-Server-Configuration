LinuxServerConfiguration
===========================================

Description
-----------

**LinuxServerConfiguration** outlines the steps involved in configuring an online Linux server to host a web application. It os the final project in the Full Stack Web Developer Nanodegree

## Information for Grader

* IP address - 54.165.131.195
* Port - 2200
* URL - ec2-54-165-131-95.compute-1.amazonaws.com


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

Next use the WSGI module to configure the apche server to handle requests ```sudo vim /etc/apache2/sites-availible/000-default.conf```

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
## Install python dev and verify WSGI is enabled

Install python-dev package ```sudo apt-get install python-dev```

Verify wsgi is enabled ```sudo a2enmod wsgi```


## Create Flask App

``` cd /var/www ```

```sudo mkdir catalog```

```cd catalog```

```sudo mkdir static templates```

```sudo vim __init__.py```
```bash

 from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, world (Testing!)"
if __name__ == "__main__":

```


## Install Flask

install flask

```sudo apt-get install python-pip```

```sudo pip install virtualenv```

```sudo virtualenv venv```

```sudo chmod -R 777 venv```

```source venv/bin/activate```

```pip install Flask```

```python __init__.py```

```deactivate```


## Configure And Enable New Virtual Host

Create host config file ```sudo vim /etc/apache2/sites-available/catalog.conf```

paste the following:

```bash
<VirtualHost *:80>
  ServerName 54.165.131.195
  ServerAdmin admin@54.165.131.195
  WSGIScriptAlias / /var/www/catalog/catalog.wsgi
  <Directory /var/www/catalog/catalog/>
      Order allow,deny
      Allow from all
  </Directory>
  Alias /static /var/www/catalog/catalog/static
  <Directory /var/www/catalog/catalog/static/>
      Order allow,deny
      Allow from all
  </Directory>
  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
    
save file

Enable `sudo a2ensite catalog`

Create the wsgi file

```cd /var/www/catalog```

```sudo vim catalog.wsgi```

```bash

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
  
application.config['DATABASE_URL'] = 'postgresql://catalog:db-password@localhost/catalog'
application.config['UPLOAD_FOLDER'] = '/var/www/catalog/catalog/item_images'
application.config['OAUTH_SECRETS_LOCATION'] = '/var/www/catalog/'
application.config['ALLOWED_EXTENSIONS'] = set(['pdf', 'jpg', 'jpeg', 'png', 'gif'])
application.config['MAX_CONTENT_LENGTH'] = 200 * 1024 * 1024  # 4 MB

# Create database and populate it, if not already done so.
create_db(application.config['DATABASE_URL'])
populate_database()

  
  ```
save file

```sudo service apache2 restart```



```mkdir catalog```
```sudo mv TrueSophia/Desktop/Sophia/vagrant/catalog/catalog/* catalog/```

Now update (appplication.py ?), fb_clients, g_clients, etc and mv to new catalog folder and check that wsgi file


## Make .git inaccessible

from ```cd /var/www/catalog/``` create .htaccess file ```sudo vim .htaccess```
paste in ```RedirectMatch 404 /\.git```
save file

## Install Dependencies 

by creating the requirements file ``` sudo vim requirements``` and runnning it ```bash requirements```

```bash

source venv/bin/activate
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install --uograde oauth2client
sudo pip install requests
sudo pip install httplib2
pip install Flask-SQLAlchemy
sudo pip install sqlalchemy
sudo pip install flask-seasurf
```


## Install and configure PostgreSQL:

Install postgresql ```sudo apt-get install postgresql```

install additional models ```sudo apt-get install postgresql-contrib```

by default ```no remote connections``` are not allowed

configure database_setup.py ```sudo vim database_setup.py```



## HERE IS WHERE THE ERROR OCCURS - IN MY CURRENT CODE I HAVE A SEPERATE FUNCTION DEFINED FOR THE URL AS WAS ORIGIANLLY FOLLOWING STEVEN WOODINGS REPO WHEREAS THE CURRENT REPO I'M FOLLOWING JUST USES THE FOLLWING LINE AND NO FUNCTION. SO MAYBE I NEED TO CHANGE THAT database_setrup.py file SO THAT IT ONLY HAS THAT LINE INSTEAD OF THE DEFINED FUNCTION? 

```python engine = create_engine('postgresql://catalog:db-password@localhost/catalog')```

repeat for application.py(main.py)

copy your main app.py file into the init.py file mv app.py __init__.py

Add catalog user ```sudo adduser catalog```

login as postgres super user ```sudo su - postgres```

enter postgresql ```psql```

Create user catalog ```CREATE USER catalog WITH PASSWORD 'db-password'```

Change role of user catalog to createDB ```ALTER USER catalog CREATEDB```

List all users and roles to verify ```\du```

Create new DB "catalog" with own of catalog ```CREATE DATABASE catalog WITH OWNER catalog```;

Connect to database ``\c catalog```

Revoke all rights ```REVOKE ALL ON SCHEMA public FROM public```;

Give accessto only to catalog role ```GRANT ALL ON SCHEMA public TO catalog```;

Quit postgressql `\q`

logout from postgresql super user ```exit```


## LOCKED OUT OF USER ACCOUNT THAT'S WHY THERE IS A FATAL ERROR FOR AUTHENTICATION
```FATAL:  password authentication failed for user "catalog"```

goto the following search page
https://www.google.ie/search?client=safari&rls=en&q=FATAL:++password+authentication+failed+for+user&ie=UTF-8&oe=UTF-8&gws_rd=cr&ei=tmnxWIXhL8vfgAbY24oo

##################################################################


Setup your database schema ```python database_setup.py```

I had problems importing psycopg2 this stack overflow post helped me

retstart apache ```sudo service apache2 restart```

I was getting a ```No such file or directory: 'client_secrets.json'``` error. I fixed using a raw path to the file ```open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())... ```You'll also need to do this for any other instances of the file path stack overflow


#########################################################################


## 11.5 - Run application

Restart Apache:
```$ sudo service apache2 restart```

Open a browser and put in your public ip-address as url, e.g. 54.165.131.195 - if everything works, the application should come up
*If getting an internal server error, check the Apache error files:
Source: A2 Hosting
View the last 20 lines in the error log: ```$ sudo tail -20 /var/log/apache2/error.log````
*If a file like 'g_client_secrets.json' couldn't been found:
Source: Stackoverflow

Could not parse file errpr

## 11.6 - Get OAuth-Logins Working

Source: Udacity and Apache

Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 54.165.131.195, its ec2-54-165-131-95.compute-1.amazonaws.com
Open the Apache configuration files for the web app: ```$ sudo vim /etc/apache2/sites-available/catalog.conf```

Paste in the following line below ServerAdmin:
```ServerAlias HOSTNAME```, e.g. ```ec2-54-165-131-95.compute-1.amazonaws.com````

Enable the virtual host:
```$ sudo a2ensite catalog```



To get the Google+ authorization working:
Go to the Developer Console: https://console.developers.google.com/ 

Click on Credentials in the sidebar and then click edit icon on the far right of the client id

add your host name and public IP-address to your Authorized JavaScript origins 

and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-52-25-0-41.us-west-2.compute.amazonaws.com/oauth2callback

To get the Facebook authorization working:
Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'11.5 - Run application

Restart Apache:
$ sudo service apache2 restart
Open a browser and put in your public ip-address as url, e.g. 52.25.0.41 - if everything works, the application should come up
*If getting an internal server error, check the Apache error files:
Source: A2 Hosting
View the last 20 lines in the error log: $ sudo tail -20 /var/log/apache2/error.log
*If a file like 'g_client_secrets.json' couldn't been found:
Source: Stackoverflow
11.6 - Get OAuth-Logins Working

Source: Udacity and Apache

Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 52.25.0.41, its ec2-52-25-0-41.us-west-2.compute.amazonaws.com
Open the Apache configuration files for the web app: $ sudo vim /etc/apache2/sites-available/catalog.conf
Paste in the following line below ServerAdmin:
ServerAlias HOSTNAME, e.g. ec2-52-25-0-41.us-west-2.compute.amazonaws.com
Enable the virtual host:
$ sudo a2ensite catalog
To get the Google+ authorization working:
Go to the project on the Developer Console: https://console.developers.google.com/project
Navigate to APIs & auth > Credentials > Edit Settings
add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-52-25-0-41.us-west-2.compute.amazonaws.com/oauth2callback
To get the Facebook authorization working:
Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'11.5 - Run application

Restart Apache:
$ sudo service apache2 restart
Open a browser and put in your public ip-address as url, e.g. 52.25.0.41 - if everything works, the application should come up
*If getting an internal server error, check the Apache error files:
Source: A2 Hosting
View the last 20 lines in the error log: $ sudo tail -20 /var/log/apache2/error.log
*If a file like 'g_client_secrets.json' couldn't been found:
Source: Stackoverflow
11.6 - Get OAuth-Logins Working

Source: Udacity and Apache

Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 52.25.0.41, its ec2-52-25-0-41.us-west-2.compute.amazonaws.com
Open the Apache configuration files for the web app: $ sudo vim /etc/apache2/sites-available/catalog.conf
Paste in the following line below ServerAdmin:
ServerAlias HOSTNAME, e.g. ec2-52-25-0-41.us-west-2.compute.amazonaws.com
Enable the virtual host:
$ sudo a2ensite catalog
To get the Google+ authorization working:
Go to the project on the Developer Console: https://console.developers.google.com/project
Navigate to APIs & auth > Credentials > Edit Settings
add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-52-25-0-41.us-west-2.compute.amazonaws.com/oauth2callback
To get the Facebook authorization working:
Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'
