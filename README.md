# Linux-Server-Configuration
Udacity Full Stack Web Developer Nanodegree Section 5 Project.

## Project Overview
In this project I create an Amazon Web Services Lightsail instance, 
start with a basic linux distribution (Ubuntu 16.04), install web and 
database services (Apache2, sqlite) and app dependencies 
(Python 2.7, Flask 1.0.2, wsgimod, and others), secure the server from a 
variety of attack vectors (update, set user permissions, configure 
firewalls (ufw and aws), change ssh port, create private / public key 
pair, and more. Finally deploy the app I made in Project 3. This step 
also involved updating google's oauth 2.0 credentials for google login
from a new public domain.

## Requirements
The grader will need the private key to log in via ssh. I will include 
the contents of the private key in the comments box when submitting my 
project for grading. The password to the private key will also be in the 
comments to grader box. In addition, he or she will need the public static 
IP, which is (23.20.77.227) and ssh port number (2200).

## Instructions
Users may now access the app I made in project 3 by visiting 
[http://glenallin.com](http://glenallin.com) The deployment of this app  
to the server I configured here for project 5 removes the previous  
requirements and instructions for setting up to run the app locally. Now 
It is as simple as navigating to the url above. 

## Instructions to grader
copy the contents of the comments to grader box and paste them into a new
file to be used as the private key. Save this file as "grader" inside .ssh 
folder. Of course the file name and location can be changed, but that will
match my example below. If you chose a different name or location just 
adjust below as needed. Finally, open your terminal of choice. I use 
gitbash. Type '''ssh -i .ssh/grader grader@23.20.77.227 -p 2200''' where 
.ssh/grader matches the location and name of the private key file you made
above.

## Software Installed and Configuration Changes Made  
Login:  
$ ssh ubuntu@23.20.77.227 -i .ssh/LightsailDefaultKey-us-east-1.pem  
  
Update:  
$ sudo apt-get update  
  
Upgrade:  
$ sudo apt-get upgrade  
  
Cleanup:  
$ sudo apt-get autoremove  
  
Install Finger:  
$ sudo apt-get install finger  
  
----------------------------------------------------------------------  
  
Add User (grader):  
$ sudo adduser --home /home/grader --shell /bin/bash --ingroup admin grader  
password: grader (will get removed later)  
  
Switch to the root user  
$ sudo su  
Add a password for that user  
$ passwd <username>  
  
Enable sudo for grader:  
$ sudo touch /etc/sudoers.d/grader  
$ sudo nano /etc/sudoers.d/grader  
Add the following line:  
"grader ALL=(ALL) ALL"  
write and quit (ctrl o, enter, ctrl x)  
  
Switch to new user:  
$ su grader  
  
Switch to the users home directory:  
$ cd /home/grader/  
  
----------------------------------------------------------------------  
  
Generate Keys:  
$ ssh-keygen -b4096 -f grader -t rsa  
password: grad3r  
  
Make a .ssh folder inside the user folder:  
$ mkdir .ssh  
  
Set permission for owner of file to read, write, and execute:  
$ chmod 700 .ssh  
  
Store public key in authorized keys file:  
$ cat grader.pub > .ssh/authorized_keys  
  
Set permission for owner to read and write to the file:  
$ chmod 600 .ssh/authorized_keys  
  
Set the owner to grader and the group to admin:  
$ sudo chown grader:admin .ssh  
  
Set the owner to grader and the group to admin:  
$ sudo chown grader:admin .ssh/authorized_keys  
  
Copy User:  
sudo rsync -avr grader /home/ubuntu/  
  
Set Permissions:  
$ sudo chmod 777 /home/ubuntu/grader  
  
---------------------- Switch to Local Terminal ----------------------   
  
Copy the key "<username>" from your AWS Server:  
scp -i .ssh/LightsailDefaultKey-us-east-1.pem    
  
ubuntu@23.20.77.227:/home/ubuntu/grader .ssh/grader  
  
Set permissions so that owner can read  
$ chmod 400 .ssh/grader  
  
----------------------------------------------------------------------  
  
Login as grader using private key:  
$ ssh -i .ssh/grader grader@23.20.77.227  
password: grad3r  
-LOGIN SUCCESSFUL!  
  
test sudo:  
$ sudo su  
[sudo] password for grader:  
grader  
-Success!  
  
----------------------------------------------------------------------  
  
Forcing Key Based Authentication: (this was already set)  
  
edit sshd_config:  
$ sudo nano /etc/ssh/sshd_config  
change "PasswordAuthentication yes" to no.  
write file and exit.  
restart to ssh service:  
$ sudo service ssh restart  
  
----------------------------------------------------------------------  
  
Remove password from grader: (this could have been done earlier when 
adding grader)  
$ sudo nano /etc/sudoers.d/grader  
change the following line from  
"grader ALL=(ALL) ALL" to "grader ALL=(ALL) NOPASSWD:ALL"  
write file and exit  
login as root:  
$ sudo su  
edit visudo:  
$ visudo  
change the following line from  
"grader ALL=(ALL) ALL" to "grader ALL=(ALL) NOPASSWD:ALL"  
write file and exit  
exit root:  
$ exit  
  
----------------------------------------------------------------------  
  
Change SSH port 22 to custom port 2200:  
add custom TCP port 2200 in AWS Lightsail Networking settings for 
23.20.77.227  
  
Configure the Uncomplicated Firewall (UFW) as follows:  
only allow incoming connections for SSH port 2200, HTTP port 80, and  
NTP port 123. (see steps below).  
  
Make sure UFW is inactive before configuring it:  
$ sudo ufw status  
Output: status: inactive  
   
Set default to deny all incoming  
$ sudo ufw default deny incoming  
  
Set default to allow all outgoing  
$ sudo ufw default allow outgoing  
   
Allow SSH:  
$ sudo ufw allow ssh  
  
Allow SSH on port 2200:  
$ sudo ufw allow 2200/tcp  
  
Allow WWW  
$ sudo ufw allow www  

Allow port 8000 (for app)  
$ sudo ufw allow 8000  

Allow Apache to redirect from 80 to 8000  
$ sudo ufw allow 'Apache Full'  
  
Enable UFW 
$ sudo ufw enable  
  
Check Status  
$ sudo ufw status  
  
Edit sshd_config  
  
$ sudo nano /etc/ssh/sshd_config  
added Port 2200 below Port 22  
write file and exit (ctrl o, enter, ctrl x)  
reboot server  
  
Login using port 2200  
$ ssh -i .ssh/grader grader@23.20.77.227 -p 2200  
Success!  
  
----------------------------------------------------------------------  
  
Now that port 2200 is working, Remove port 22  
  
$ sudo nano /etc/ssh/sshd_config  
Remove Port 22  
write file and exit (ctrl o, enter, ctrl x)  

Remove SSH Port 22 from AWS Lightsail Networking settings.
Reboot server

$ sudo ufw allow ntp
$ sudo ufw deny 22
  
Login using port 2200  
$ ssh -i .ssh/grader grader@23.20.77.227 -p 2200  
Success!  

----------------------------------------------------------------------  
  
Install Apache
$ sudo apt-get install apache2

Now going to 23.20.77.227 in a browser shows the Apache2 landing page.
  
----------------------------------------------------------------------  
  
Install mod_wsgi to serve Python 2.7  
$ sudo apt-get install libapache2-mod-wsgi  

skipping testing that was done here as it is long and not necessary for 
the end result. You can see everything I did in the notes files.
  
----------------------------------------------------------------------  
  
Install PostgreSQL  
$ sudo apt-get install postgresql postgresql-contrib  
Login to postgresql as default user
$ sudo -u postgres psql
Create catalog user
postgres=# CREATE USER catalog WITH PASSWORD ‘mypassword’;
postgres=# ALTER USER catalog CREATEDB;
Create catalog database
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
  
----------------------------------------------------------------------  
  
Install dependencies:  
$ sudo apt install python-pip  
$ pip install Flask  
$ pip install sqlalchemy  
$ pip install oath2client  
$ pip install requests  
  
----------------------------------------------------------------------  
  
Setup venv virtual environment:  
$ cd /var/www/FlaskApp  
$ sudo pip install virtualenv  
$ sudo virtualenv venv  
$ source venv/bin/activate  
$ sudo pip install Flask  
$ pip install sqlalchemy  
$ pip install oauth2client  
$ pip install requests 
  
----------------------------------------------------------------------  
  
Install git  
$ sudo apt-get install git  
  
Configure git for first time use:  
$ git config --global user.name "G1enB1and"  
$ git config --global user.email "glen.bland.81@gmail.com"  
$ git config --global color.ui auto  
$ git config --global merge.conflictstyle diff3  
$ git config --list  
  
Clone my git repository for project 3's catalog app:  
$ sudo mkdir /home/ubuntu/gitrepo
$ cd /home/ubuntu/gitrepo
$ sudo git clone https://github.com/G1enB1and/fullstack-nanodegree-vm catalog

Move, rename, or delete default and test pages.  
$ cd /var/www/html  
$ sudo mv index.html apache2index.html  
$ sudo mv myapp.wsgi helloworld.wsgi  

Move files so new location is not a git repository  
$ sudo mv /home/ubuntu/gitrepo/catalog/vagrant/catalog /var/www/FlaskApp/FlaskApp/  
  
----------------------------------------------------------------------  
  
Create flaskapp.wsgi  
$ sudo touch /var/www/FlaskApp/flaskapp.wsgi  
$ sudo nano /var/www/FlaskApp/flaskapp.wsgi  
  
#!/usr/bin/python

import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = "redacted"
  
write file and exit (ctrl o, enter, ctrl x)  
  
----------------------------------------------------------------------  
  
Create FlaskApp.conf in sites-available  
$ cd /etc/apache2/sites-available/  
$ sudo touch FlaskApp.conf  
$ sudo nano FlaskApp.conf  
  
<VirtualHost *:80>
		ServerName glenallin.com
		ServerAdmin glen@glenbland.com

		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi

		<Directory /var/www/FlaskApp/FlaskApp>
			Require all granted
		</Directory>

		Alias /static /var/www/FlaskApp/FlaskApp/static

		<Directory /var/www/FlaskApp/FlaskApp/static>
			Require all granted
		</Directory>

		ErrorLog /var/www/FlaskApp/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
  
Write file and exit (ctrl o, enter, ctrl x)  
Restart apache2  
$ sudo service apache2 restart  
  
----------------------------------------------------------------------  
  
Enable site  
$ sudo a2ensite catalog  

Enable Proxy  
$ sudo a2enmod proxy && sudo a2enmod proxy_http  
  
Disable default site  
$ sudo a2dissite 000-default  
$ sudo systemctl restart apache2  
  
Check Apache files for any syntax errors:  
$ sudo apache2ctl configtest  
Syntax OK  
  
Restart server  
$ sudo systemctl restart apache2  
$ sudo apache2ctl restart  
  
----------------------------------------------------------------------  
  
Set Permissions  
$ sudo chmod -R 755 /var/www  
$ sudo chmod 755 /var/www/FlaskApp/venv/  
and all other directories
$ sudo chmod 756 /var/www/FlaskApp/FlaskApp/static/us3r_pic_1.jpg
and all other files

$ sudo chown www-data:www-data all files and directories in project
  
----------------------------------------------------------------------  
  
Edit python code in the following ways:  
Change sqlite to postgresql in engine for __init__.py, database_setup.py, and populate_catalog.py  
Change relative paths to full path in unix form.  
This had to be done for path to database, user pic location, and client_secrets.json  
Change all instances of user/User to something else, because that is reserved for psql and will cause errors.  
I used us3r instead.  
this also needed to be done in show-item.html and for us3r_pic_1.jpg for consistency.
  
----------------------------------------------------------------------  
  
Web App is now fully functional and can be accessed by browsing to glenallin.com  
  
----------------------------------------------------------------------  
  
## Third Party Resources Used  
I registered glenallin.com with Dreamhost,
created a DNS Zone in AWS Lightsail,
used the following Nameservers provided by my Lightsail DNS Zone:  
ns-493.awsdns-61.com  
ns-1865.awsdns-41.co.uk  
ns-1014.awsdns-62.net  
ns-1419.awsdns-49.org  
instead of the default nameservers in Dreamhost.  
  
Added a record in the Lightsail DNS records to point @.glenallin.com to 
Udacity-StaticIp-Virginia-1 (My Lightsail Instance)
note that @. in the subdomain field means no subdomain, but rather 
the main domain glenallin.com.  
  
I also used the Google Developer Console to provide my app with oauth 2.0 
login via google functionality. I added glenallin.com as my top level domain
for oauth credentials and added http://glenallin.com/gconnect and
http://glenallin.com/login under authorized redirects. I saved my new 
client_secrets.json file to the appropriate directory on the server with 
my app.  
  
In the AWS Lightsail Networking tab under firewall I added
custom tcp port 2200 and 8000 and removed ssh 22 (after setting up 2200)  
  
## License  
MIT Licesne found [here](LICENSE.md)  