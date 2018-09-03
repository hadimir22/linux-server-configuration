#Udacity - Linux Server Configuration Project
this is for  Udacity Full Stack Web Developer II Nanodegree Project.

#TL;DR

1. open git bash
2. save the SSH key in your local machene
3. type  ssh grader@138.68.250.33 -p 2200
4. enter "password" as passphrase
5. type cd /var/www/FlaskApp/FlaskApp
6. type source venv/bin/activate
7. run python __init__.py
8. goto browser and type  http://138.68.250.33.xip.io

>SSH key
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDC6Rj2OLX9tTXJN7BTzrAUiy7CnLRgRX8Lh55PxnAoo2p0pofJvgFlNJnqrO8Qhg5+A+AqvzGNwNZFQVpz96KZxX6qELBc8iM0Q0MKh7BtDiyafPPQC99rZiMVlCjyEjHBdrMhFjnpte8/Xso6o2um/On5ijwnTE8kYYIQbJddo1PJt64ilSwwGBN7BmK7/8AxK/odnpaRvhJvySQZ8K60fItf01xOQSXRieY4EaTCss+MJiacrJG3GtycTVbsJccuFCDZGZGQVuUgC7BjiuiCKQuvvVTURt0zkqBnr3UgO6BHyzNqBDOFmHZvR0qbBudossykG7nu0dWRtKPtaINf hadimir@hadi


#About
This tutorial will guide you through the steps to take a baseline installation of a Linux server and prepare it to host your Web applications. You will then secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing Flask-based Web applications onto it.

In this project, I have set up an Ubuntu 18.04 image on a DigitalOcean droplet. The technical details of the server as well as the steps that have been taken to set it up can be found in the succeeding sections.

#Information About the Project
Server IP Address: 138.68.250.33
SSH server access port: 2200
SSH login username: grader
password : grader
Application URL: http://138.68.250.33.xip.io
SHA key passphrase : password




> to login the server type  ssh grader@138.68.250.33 -p 2200 and enter the passphrase for key which is 'password'


#Steps to Set up the Server
> Creating the RSA Key Pair
On your local machine, you will first have to set up the public and private key pair. This key pair will be used to authenticate yourself while securely logging in to the server via SSH. The private key will be kept with you in your local machine, and the public key will be stored in the server.

To generate a key pair, open git bash and run the following command:

$ ssh-keygen
When it asks to enter a passphrase, you may either leave it empty or enter some passphrase(i have used 'password' as a passphrase . A passphrase adds an additional layer of security to prevent unauthorized users from logging in.

Then it will generate 2 files the one with the extension .pub is uploaded on server


> Setting Up a DigitalOcean Droplet
Log in or create an account on DigtalOcean.

Go to the Dashboard, and click Create Droplet.

Choose Ubuntu 18.04 x64 image from the list of given images.

Choose a preferred size. In this project, I have chosen the 1GB/1 vCPU/25GB configuration.

In the section Add Your SSH Keys, paste the content of your public key

Add SSH Keys image

This step will automatically create the file ~/.ssh/authorized_keys with appropriate permissions and add your public key to it. It would also add the following rule in the /etc/ssh/sshd_config file automatically:

>Enforce Key-based SSH authentication 
PasswordAuthentication no
This rule essentially disables password authentication on the root user, and rather enforces SSH logins only.

Click Create to create the droplet. This will take some time to complete. After the droplet has been created successfully, a public IP address will be assigned. In this project, the public IPv4 address that I have been assigned is 138.68.250.33

>Logging In as root via SSH and Updating the System
Logging in as root via SSH
As the droplet has been successfully created, you can now log into the server as root user by running the following command in your host machine:

  $ ssh root@138.68.250.33
This will look for the private key in your local machine and log you in automatically if the corresponding public key is found on your server. After you are logged in, you might see something similar to this:

Root login

>updating system packages
Run the following command to update the virtual server:

 # apt update && apt upgrade
This will update all the packages. If the available update is a kernel update, you might need to reboot the server by running the following command:

# reboot

>Changing the SSH Port from 22 to 2200
Open the /etc/ssh/sshd_config file with nano or any other text editor of your choice:

# nano /etc/ssh/sshd_config
Find the line #Port 22 (would be located around line 13) and change it to Port 2200, and save the file.

Restart the SSH server to reflect those changes:

# service ssh restart
To confirm whether the changes have come into effect or not, run:

# exit
This will take you back to your host machine. After you are back to your local machine, run:

$ ssh root@138.68.250.33 -p 2200
You should now be able to log in to the server as root on port 2200. The -p option explicitly tells at what port the SSH server operates on. It now no more operates on port number 22.

 
> Setting Up the Firewall
Now we would configure the firewall to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):

# ufw allow 2200/tcp
# ufw allow 80/tcp
# ufw allow 123/udp
To enable the above firewall rules, run:

# ufw enable
To confirm whether the above rules have been successfully applied or not, run:

# ufw status
You should see something like this:

Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)

> Creating the User grader and Adding it to the sudo Group
While being logged into the virtual server, run the following command and proceed:

# adduser grader
The output would look like this:

  Adding user `grader' ...
  Adding new group `grader' (1000) ...
  Adding new user `grader' (1000) with group `grader' ...
  Creating home directory `/home/grader' ...
  Copying files from `/etc/skel' ...
  Enter new UNIX password:
  Retype new UNIX password:
  passwd: password updated successfully
  Changing the user information for grader
  Enter the new value, or press ENTER for the default
	  Full Name []: Grader
	  Room Number []:
	  Work Phone []:
	  Home Phone []:
	  Other []:
  Is the information correct? [Y/n]
Note: Above, the UNIX password I have entered for the user grader is, grader.

>Adding grader to the Group sudo
Run the following command to add the user grader to the sudo group to grant it administrative access:

# usermod -aG sudo grader

> Adding SSH Access to the user grader
To allow SSH access to the user grader, first log into the account of the user grader from your virtual server:

# su - grader
 
Now enter the following commands to allow SSH access to the user grader:

$ mkdir .ssh
$ chmod 700 .ssh
$ cd .ssh/
$ touch authorized_keys
$ chmod 644 authorized_keys
After you have run all the above commands, go back to your local machine and copy the content of the public key file ~/.ssh/id_rsa.pub. Paste the public key to the server's authorized_keys file using nano or any other text editor, and save.

After that, run exit. You would now be back to your local machine. To confirm that it worked, run the following command in your local machine:
 
  ssh grader@138.68.250.33 -p 2200

You should now be able to log in as grader and would get a prompt to enter commands.

Next, run exit to go back to the host machine and proceed to the following step to disable root login.

>Disabling Root Login
Run the following command on your local machine to log in as root in the server:

$ ssh root@138.68.250.33 -p 2200
After you are logged in, open the file /etc/ssh/sshd_config with nano:

# nano /etc/ssh/sshd_config
Find the line PermitRootLogin yes and change it to PermitRootLogin no.

Restart the SSH server:

# service ssh restart
Terminate the connection:

# exit
After you are back to the host machine, when you try to log in as root, you should get an error:
 
  ssh root@138.68.250.33 -p 2200
root@138.68.250.33: Permission denied (publickey).

>Installing Apache Web Server
To install the Apache Web Server, run the following command after logging in as the grader user via SSH:

$ sudo apt update
$ sudo apt install apache2
To confirm whether it successfully installed or not, enter the URL http://138.68.250.33 in your Web browser:

If the installation has succeeded, you should see the default apache page
 
>Installing pip3
The package pip3 will be required to install certain packages. To install it, run:

$ sudo apt install python3-pip
To confirm whether or not it has been successfully installed, run:

$ pip3 --version
You should see something like this if it has been successfully installed:

pip 9.0.1 from /usr/lib/python3/dist-packages (python 3.6)

> Installing and Configuring Git
12.1. Installing Git
In Ubuntu 18.04, git might already be pre-installed. If it isn't, run the following commands:

$ sudo add-apt-repository ppa:git-core/ppa
$ sudo apt update
$ sudo apt install git

> Configuring Git
To continue using git, you will have to configure a username and an email:

$ git config --global user.name "hadimir22"

$ git config --global user.email "hadimir22@gmail.com"

>Installing PostgreSQL
Install PostgreSQL:

$ sudo apt install postgresql

>Configuring PostgreSQL

Log in as the user postgres that was automatically created during the installation of PostgreSQL Server:

$ sudo su - postgres
Open the psql shell:

$ psql
This will open the psql shell. Now type the following commands one-by-one:

postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
Then exit from the terminal by running \q followed by exit.

>Setting Up Apache to Run the Flask Application
 Installing mod_wsgi
The module mod_wsgi will allow your Python applications to run from Apache server. To install it, run the following command:

$ sudo apt install libapache2-mod-wsgi-py3
This would also enable wsgi. So, you don't have to enable it manually.

After the installation has succeeded, restart the Apache server

$ sudo service apache2 restart

> Cloning the Item Catalog Flask application
Change the current working directory to /var/www/:

$ cd /var/www/
Create a directory called FlaskApp and change the working directory to it:

$ sudo mkdir FlaskApp
$ cd FlaskApp/
Clone this repository as the directory FlaskApp:

$ sudo git clone https://github.com/hadimir22/item-catalog.git FlaskApp
Move inside the newly created directory:

$ cd FlaskApp/ 
 

The directory tree should now look like this:

*install tree to display directory like this and then type tree -L 3* 

 FlaskApp
    ├── FlaskApp
    │   ├── client_secrets.json
    │   ├── database_setup.py
    │   ├── database_setup.pyc
    │   ├── fakeitems.py
    │   ├── fb_client_secrets.json
    │   ├── __init__.py
    │   ├── item-catalog
    │   ├── __pycache__
    │   ├── README.md
    │   ├── static
    │   ├── templates



> Installing virtualenv and All the Required Packages
To install virtualenv, run the following command:

$ sudo pip3 install virtualenv

Then move to /var/www/FlaskApp/FlaskApp:

$ cd /var/www/FlaskApp/FlaskApp

Create a Virtual Environment:

$ sudo python3 -m virtualenv venv

Change the mode of venv to 777:

$ sudo chmod 777 venv/

Activate venv:

$ source venv/bin/activate

You should now see a prompt like this:

(venv) grader@fsnd:/var/www/FlaskApp/FlaskApp$

Install required packages:

$ sudo pip3 install --upgrade Flask SQLAlchemy httplib2 oauth2client requests psycopg2 psycopg2-binary

> Setting Up Virtual Hosts

Run the following command in terminal to set up a file called FlaskApp.conf to configure the virtual hosts:

$ sudo nano /etc/apache2/sites-available/FlaskApp.conf
Add the following lines to it:


<VirtualHost *:80>
   ServerName 138.68.250.33
   ServerAlias 138.68.250.33.xip.io
   ServerAdmin hadimir22@gmail.com
   WSGIDaemonProcess FlaskApp python-path=/var/www \
     python-home=/var/www/FlaskApp/venv
   WSGIProcessGroup FlaskApp
   WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
   <Directory /var/www/FlaskApp/FlaskApp/>
       Require all granted
   </Directory>
   Alias /static /var/www/FlaskApp/FlaskApp/static
   <Directory /var/www/FlaskApp/FlaskApp/static/>
       Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   LogLevel warn
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


Enable the virtual host:

$ sudo a2ensite FlaskApp

Restart Apache server:

$ sudo service apache2 restart

>Creating the .wsgi File

Apache uses the .wsgi file to serve the Flask app. Move to the /var/www/FlaskApp/ directory and create a file named flaskapp.wsgi with following commands:

$ cd /var/www/FlaskApp/
$ sudo nano flaskapp.wsgi

Add the following lines to the flaskapp.wsgi file:

#!/var/www/FlaskApp/venv/bin/python3
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'

Restart Apache server:

$ sudo service apache2 restart
Now you should be able to run the application at http://138.68.250.33.xip.io/.


Note: You might still see the default Apache page despite setting everything up correctly. To resolve it and see your Flask app running, run the following commands in order:

$ sudo a2dissite 000-default.conf
$ sudo service apache2 restart

>Debugging
If you are getting an Internal Server Error or any other error(s), make sure to check out Apache's error log for debugging:

$ sudo cat /var/log/apache2/error.log

>References
[1] https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04

[2] http://terokarvinen.com/2016/deploy-flask-python3-on-apache2-ubuntu

[3] https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

>note
1. you might need to install the liabraries/pakages/dependencies in "(venv) grader@fsnd:/var/www/FlaskApp/FlaskApp$" also.
2. run python database_setup.py and python fakeitems.py to populate database
3. the aap logic should be in __init__.py and then run python __init__.py 
4. If you want to see what packages have been installed with your installer tools : pip freeze

