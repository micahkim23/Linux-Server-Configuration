# FSND-P5_Linux-Server-Configuration

A installation of Ubuntu Linux on a virtual machine to host a Flask web application. This includes the installation of updates, a firewall to secure the system from attack vectors, and web and database servers.

This is a walkthrough to project 5 of Udacity's Full Stack Web Developer Nanodegree and deploys project 3 on the virtual machine.


## 1 & 2 Create Development Environment: Launch Virtual Machine and SSH into server

Source: [Udacity](https://www.udacity.com/account#!/development_environment)

1. Open up terminal
2. Download private key and take note of public ip address
3. Move the private key file into the folder ~/.ssh (where ~ is the environment's home directory). If you downloaded the file to the Downloads folder, execute the following command in your terminal.
'''$ mv ~/Downloads/udacity_key.rsa ~/.ssh/'''
4. Open your terminal and type in
'''$ chmod 600 ~/.ssh/udacity_key.rsa'''
5. In the terminal type in
'''$ ssh -i ~/.ssh/udacity_key.rsa root@PUBLIC-IP-ADDRESS'''

## 3 & 4 Create New User and Give User Permission to Sudo

Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

1. Create a new user:
'''$ adduser newuser'''
2. Give user permission to sudo:
	i. Open up visudo (user configuration file):
	'''$ visudo'''
	ii. Search for the line that says:
	'''root ALL=(ALL:ALL) ALL'''
	iii. Below that line type:
	'''newuser ALL=(ALL:ALL) ALL'''

## 5 Update all currently installed packages:

Source: [Udacity Configuring Linux Web Servers Course](https://www.udacity.com/course/configuring-linux-web-servers--ud299-nd)

1. Update available packages and versions:
'''$ sudo apt-get update'''
2. Install newer versions of packages:
'''$ sudo apt-get upgrade'''

## 6 Change SSH port from 22 to 2200 and configure SSH access

Source: [Udacity Configuring Linux Web Servers Course](https://www.udacity.com/course/configuring-linux-web-servers--ud299-nd)

1. Change ssh config file:
	i. Open the config file:
	'''$ nano /etc/ssh/sshd_config'''
	ii. Search for the line that says:
	'''port 22'''
	Change that line to:
	'''port 2200'''
	iii. Search for the line that says:
	'''PasswordAuthentication no'''
	Change that line to:
	'''PasswordAuthentication yes'''
	(note: temporarily allow PasswordAuthentication)
	iv. Change PermitRootLogin to no:
	'''PermitRootLogin no'''
	v. Append:
	'''AllowUsers newuser'''
	vi. reload the configuration:
	'''$ service ssh reload'''
	vii. Open a new terminal window and login in as newuser with password
	'''$ ssh newuser@PUBLIC-IP-ADDRESS'''
2. Create SSH KEY:
	i. Generate a SSH key pair on the local machine in a new terminal window:
	'''$ ssh-keygen'''
	(note: the default key files are id_rsa and id_rsa.pub)
3. Make sure newuser has public SSH KEY:
	i. In the terminal window that you were logged in as newuser, create a new directory called .ssh:
	'''$ mkdir .ssh'''
	ii. create an empty authorized_keys file inside of .ssh:
	'''$ touch .ssh/authorized_keys'''
	iii. in a new terminal window, copy contents of ~/.ssh/id_rsa.pub:
	'''$ cat .ssh/id_rsa.pub'''
	iv. paste contents of ~/.ssh/id_rsa.pub into newuser's authorized_keys file:
	'''$ nano .ssh/authorized_keys'''
	v. change file permissions:
	'''$ chmod 700 .ssh'''
	'''$ chmod 644 .ssh/authorized_keys'''
4. login as newuser using SSH KEY:
	i. '''$ ssh newuser@PUBLIC-IP-ADDRESS -p 2200 -i ~/.ssh/id_rsa'''
5. Change Password Authentication back to no:
	i. while logged as newuser, type:
	''' $ sudo nano /etc/ssh/sshd_config'''
	ii. change password Authentication:
	PasswordAuthentication no
	iii. reload the configuration:
	''' $ sudo service ssh reload'''

## 7 Configure the Uncomplicated Firewall(UFW) to only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

Source: [Udacity Configuring Linux Web Servers Course](https://www.udacity.com/course/configuring-linux-web-servers--ud299-nd)

1. Deny all incoming connections:
''' $ sudo ufw default deny incoming'''
2. Allow all outgoing connections:
''' $ sudo ufw default allow outgoing'''
3. Allow incoming TCP packets on port 2200 (SSH):
''' $ sudo ufw allow 2200/tcp'''
4. Allow incoming TCP pakcets on port 80 (HTTP):
''' $ sudo ufw allow 80/tcp'''
5. Allow incoming UDP packets on port 123 (NTP):
''' $ sudo ufw allow 123/udp'''
6. enable firewall:
''' $ sudo ufw enable'''
7. check status of firewall:
''' $ sudo ufw status'''

## 8 Configure local timezone to UTC

Source: [Ubuntu documentation](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)

1. open timezone selection dialog
'''$ sudo dpkg-reconfigure tzdata'''
2. Choose 'none of the above', then select UTC

## 9 Install and configure Apache to serve a Python mod_wsgi application

Source: [Udacity Configuring Linux Web Servers Course](https://www.udacity.com/course/configuring-linux-web-servers--ud299-nd)

1. Install Apache web server:
'''$ sudo apt-get install apache2'''
2. Open a browser at your public ip address, http://54.200.104.123, and it should open up the Apache2 Ubuntu Default Page
3. Install the mod_wsgi tool:
'''$ sudo apt-get install python-setuptools libapache2-mod-wsgi'''
4. Restart Apache server, so that mod_wsgi can load:
'''$ sudo service apache2 restart'''

## 10 Install and configure Postgresql

1. To install PostgreSQL, type:
'''$ sudo apt-get install postgresql'''
2. Check that it doesn't allow remote connections:
'''$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf'''

## 11 Install git, clone and setup Catalog App project

### 11.1 Install git

Source: [Github](https://help.github.com/articles/set-up-git/#platform-linux)
1. Install Git:
'''$ sudo apt-get install git'''
2. Set your name:
'''$ git config --global user.name "YOUR NAME"'''
3. Set your email:
'''$ git config --global user.email ""YOUR EMAIL ADDRESS"'''

### 11.2 - Setup for deploying a Flask Application on Ubuntu VPS
Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps "How To Deploy a Flask Application on an Ubuntu VPS")

1. Extend Python with additional packages that enable Apache to serve Flask applications:  
  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable mod_wsgi (if not already enabled):  
  `$ sudo a2enmod wsgi`
3. Create a Flask app:  
  i. Move to the www directory:  
    `$ cd /var/www`
  ii. Setup a directory for the app, e.g. catalog:  
    a. `$ sudo mkdir FlaskApp`  
    b. `$ cd FlaskApp` and `$ sudo mkdir FlaskApp`  
    c. `$ cd FlaskApp` and `$ sudo mkdir static templates`  
    d. Create the file that will contain the flask application logic:  
      `$ sudo nano __init__.py`
    e. Paste in the following code:  
    ```python  
      from flask import Flask  
      app = Flask(__name__)  
      @app.route("/")  
      def hello():  
        return "Hello, I love Digital Ocean!"  
      if __name__ == "__main__":  
        app.run()  
    ```  
4. Install Flask
  i. Install pip installer:  
    `$ sudo apt-get install python-pip` 
  ii. Install virtualenv:  
    `$ sudo pip install virtualenv`
  iii. Set virtual environment to name 'venv':  
    `$ sudo virtualenv venv`
  iv. Activate the virtual environment:  
    `$ source venv/bin/activate`
  v. Install Flask inside the virtual environment:  
    `$ pip install Flask`
  vi. Run the app:  
    `$ python __init__.py`
  vii. Deactivate the environment:  
    `$ deactivate`
5. Configure and Enable a New Virtual Host#
  i. Create a virtual host config file  
    `$ sudo nano /etc/apache2/sites-available/FlaskApp.conf`
  ii. Paste in the following lines of code and change names and addresses regarding your application:  
  ```
    <VirtualHost *:80>
        ServerName PUBLIC-IP-ADDRESS
        ServerAdmin admin@PUBLIC-IP-ADDRESS
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
  iii. Enable the virtual host:  
    `$ sudo a2ensite FlaskApp`
6. Create the .wsgi File and Restart Apache
  i. Create wsgi file:  
    `$ cd /var/www/FlaskApp` and `$ sudo nano FlaskApp.wsgi`
  ii. Paste in the following lines of code:  
  ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")
    
    from catalog import app as application
    application.secret_key = 'Add your secret key'
  ```
  7. Restart Apache:  
    `$ sudo service apache2 restart`