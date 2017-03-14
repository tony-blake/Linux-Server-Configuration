LinuxServerConfiguration
===========================================

Description
-----------

**LinuxServerConfiguration** outlines the steps involved in configuring an online Linux server to host a web application. It os the final project in the Full Stack Web Developer Nanodegree

## Information for Grader

* IP address - 54.68.3.20
* Port - 2200
* URL -


## Obtain access to virtual server

Sign into your udacity account and go to this part of the website https://www.udacity.com/account#!/development_environment.
Click on create development environment.
Click on the download link to obtain private key.
Then move private key to .ssh folder.
Log into server.

```bash
mv ~/Downloads/udacity_key.rsa ~/.ssh/
chmod 600 ~/.ssh/udacity_key.rsa
ssh -i ~/.ssh/udacity_key.rsa root@54.68.3.20

```

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
The file opnes in nano and you can insert the line ```grader ALL=(ALL:ALL) ALL ``` under  ``` "#User privilege specification" ``` underneath ```root ALL=(ALL:ALL) ALL ``` . Then save file by hitting ``` ctrl + x ``` and selecting Yes.

## unable to resolve host warning

When Running the sudo command initially a warning appears.  This is because the host ip address is not listed in the ``` /etc/hosts ``` file

```bash
root@ip-10-20-41-235:~# sudo visudo
sudo: unable to resolve host ip-10-20-41-235

```
To address the complaint the host ip address (in this case ``` ip-10-20-41-235 ```) needs to be added to top of the ``` /etc/hosts ``` file in the following way

```bash
127.0.0.1 localhost ip-10-20-41-235 
```

## Secure the server

To update all packages use

```bash
sudo apt-get update
sudo apt-get upgrade
```
## Change the SSH port from 22 to 2200 and reconfigure authetications
```bash
vim /etc/ssh/sshd_config
```
Then change ```Port 22``` to ```Port 2200```
Change ```PasswordAuthetication yes``` to ```PasswordAuthetication no```
And change ```PermitRootLogin without-password``` to ```PermitRootLogin no```
Then append ```AllowUsers grader``` to end of file
Lastly run ```sudo service ssh restart``` for changes to take effect

## Create SSH keys and copy to server.

On local machine generate SSH key pair by running command ``` ssh-keygen ```

```bash
Macintosh-109add6f31eb:~ tonyblake$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/tonyblake/.ssh/id_rsa): /Users/tonyblake/.ssh/id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/tonyblake/.ssh/id_rsa.
Your public key has been saved in /Users/tonyblake/.ssh/id_rsa.pub.
The key fingerprint is:
f4:61:fa:47:91:7a:86:e8:72:29:0a:85:86:ee:e9:da tonyblake@Macintosh-109add6f31eb.local
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|             .   |
|        . o o    |
| . .   . = + .   |
|. o .   S + +    |
|.. .   . o +     |
| ..   o + . .    |
|.... . +   .     |
|++E .            |
+-----------------+
Macintosh-109add6f31eb:~ tonyblake$ 
```

Then log into grader account ```ssh -v grader@54.68.3.20 -p 2200 ```
where password is "Passw0rd" 

## Authorize and set permssions

Create .ssh directory ```mkdir .ssh```
Create empty file to store key ``` bash .ssh/authorized_keys ```
On local machine read contents of the public key ```cat .ssh/rsa_id.pub```
Copy the key and paste into the authorized_keys file in grader ```vim .ssh/authorized_keys``` 
Set permissions for files: ```chmod 700 .ssh```  and ```chmod 644 .ssh/authorized_keys```
Change ```PasswordAuthentication``` from ```yes``` back to ```no```. vim /etc/ssh/sshd_config
save file(nano: ctrl+x, Y, Enter)
login with key pair: ```ssh -v grader@54.68.3.20 -p 2200 -i ~/.ssh/id_rsa```



```bash
grader@ip-10-20-36-35:~$ mkdir .ssh
grader@ip-10-20-36-35:~$ vim .ssh/authorized_keys
grader@ip-10-20-36-35:~$ vim .ssh/authorized_keys
grader@ip-10-20-36-35:~$ chmod 700 .ssh
grader@ip-10-20-36-35:~$ chmod 644 .ssh/authorized_keys
grader@ip-10-20-36-35:~$ vim /etc/ssh/sshd_config
grader@ip-10-20-36-35:~$ sudo vim /etc/ssh/sshd_config
[sudo] password for grader: 
grader@ip-10-20-36-35:~$ exit
logout
```

## Configure Firewalls

Check UFW status to make sure its inactive ```sudo ufw status```
Deny all incoming by default ```sudo ufw default deny incoming```
Allow outgoing by default ```sudo ufw default allow outgoing```
Allow SSH on port 2200 ```sudo ufw allow 2200/tcp```
Allow HTTP on port 80 ``` sudo ufw allow 80/tcp```
Allow NTP on port 123 ```sudo ufw allow 123/udp ```
Turn on firewall ```sudo ufw enable```

The terminal output should look like this

```bash
grader@ip-10-20-36-35:~$ sudo ufw status
[sudo] password for grader: 
Status: inactive
grader@ip-10-20-36-35:~$ sudo ufw default deny incoming
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)
grader@ip-10-20-36-35:~$ sudo ufw default allow outgoing
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)
grader@ip-10-20-36-35:~$ sudo ufw allow 2200/tcp
Rules updated
Rules updated (v6)
grader@ip-10-20-36-35:~$ sudo ufw allow 80/tcp
Rules updated
Rules updated (v6)
grader@ip-10-20-36-35:~$ sudo ufw allow 123/udp
Rules updated
Rules updated (v6)
grader@ip-10-20-36-35:~$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
grader@ip-10-20-36-35:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)

grader@ip-10-20-36-35:~$ 





```

## Configure the local timezone to UTC
 
run ```sudo dpkg-reconfigure tzdata```
Then when prompted select "none of the above". 
Then select "UTC".

## Install and configure Apache to serve a Python mod_wsgi application

run ``` sudo apt-get install apache2``` 
Type public ip address (54.68.3.20) into URL and check if apache webpage is there (sceenshot)
install mod_wsgi: ```sudo apt-get install libapache2-mod-wsgi```
Next use the WSGI module to configure the apche server to handle requests ```sudo vim /etc/apache2/sites-enabled/000-default.conf```
Add ```WSGIScriptAlias / /var/www/html/myapp.wsgi``` before ```</ VirtualHost>```
Restart Apache ```sudo apache2ctl restart```

## Restart Error

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
```
Install python dev and verify WSGI is enabled

```sudo apt-get install python-dev
   sudo a2enmod wsgi```
   




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



