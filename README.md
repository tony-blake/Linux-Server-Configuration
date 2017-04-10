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

Create host config file ``sudo vim /etc/apache2/sites-available/catalog.conf```

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
  
  ```
save file

```sudo service apache2 restart```



```mkdir catalog```
```sudo mv TrueSophia/Desktop/Sophia/vagrant/catalog/* catalog/```

Now update (appplication.py ?), fb_clients, g_clients, etc and mv to new catalog folder and check that wsgi file
