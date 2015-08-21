# FSND-P5_Linux-Server-Configuration

A installation of Ubuntu Linux on a virtual machine to host a Flask web application. This includes the installation of updates, a firewall to secure the system from attack vectors, and web and database servers.

This is a walkthrough to project 5 of Udacity's Full Stack Web Developer Nanodegree and deploys project 3 on the virtual machine.


## 1 & 2 Create Development Environment: Launch Virtual Machine and SSH into server

Source: [Udacity](https://www.udacity.com/account#!/development_environment)

1. Open up terminal
2. Download private key and take note of public ip address
3. Move the private key file into the folder ~/.ssh (where ~ is the environment's home directory). If you downloaded the file to the Downloads folder, execute the following command in your terminal.
  `$ mv ~/Downloads/udacity_key.rsa ~/.ssh/`
4. Open your terminal and type in
  `$ chmod 600 ~/.ssh/udacity_key.rsa`
5. In the terminal type in
  `$ ssh -i ~/.ssh/udacity_key.rsa root@PUBLIC-IP-ADDRESS`

## 3 & 4 Create New User and Give User Permission to Sudo

Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

1. Create a new user:
'''$ adduser newuser'''
2. Give user permission to sudo:
	1. Open up visudo (user configuration file):
	  `$ visudo`
	2. Search for the line that says:
	  `root ALL=(ALL:ALL) ALL`
	3. Below that line type:
	  `newuser ALL=(ALL:ALL) ALL`

## 5 Update all currently installed packages:

Source: [Udacity Configuring Linux Web Servers Course](https://www.udacity.com/course/configuring-linux-web-servers--ud299-nd)

1. Update available packages and versions:
  `$ sudo apt-get update`
2. Install newer versions of packages:
  `$ sudo apt-get upgrade`

## 6 Change SSH port from 22 to 2200 and configure SSH access

Source: [Udacity Configuring Linux Web Servers Course](https://www.udacity.com/course/configuring-linux-web-servers--ud299-nd)

1. Change ssh config file:
	1. Open the config file:
	  `$ nano /etc/ssh/sshd_config`
	2. Search for the line that says:
	  `port 22`
	Change that line to:
	  `port 2200`
	3. Search for the line that says:
	  `PasswordAuthentication no`
	Change that line to:
	  `PasswordAuthentication yes`
	(note: temporarily allow PasswordAuthentication)
	4. Change PermitRootLogin to no:
	  `PermitRootLogin no`
	5. Append:
	  `AllowUsers newuser`
	6. reload the configuration:
	  `$ service ssh reload`
	7. Open a new terminal window and login in as newuser with password
	  `$ ssh newuser@PUBLIC-IP-ADDRESS`
2. Create SSH KEY:
	1. Generate a SSH key pair on the local machine in a new terminal window:
	  `$ ssh-keygen`
	(note: the default key files are id_rsa and id_rsa.pub)
3. Make sure newuser has public SSH KEY:
	1. In the terminal window that you were logged in as newuser, create a new directory called .ssh:
	  `$ mkdir .ssh`
	2. create an empty authorized_keys file inside of .ssh:
	  `$ touch .ssh/authorized_keys`
	3. in a new terminal window, copy contents of ~/.ssh/id_rsa.pub:
	  `$ cat .ssh/id_rsa.pub`
	4. paste contents of ~/.ssh/id_rsa.pub into newuser's authorized_keys file:
	  `$ nano .ssh/authorized_keys`
	5. change file permissions:
	  `$ chmod 700 .ssh`
	  `$ chmod 644 .ssh/authorized_keys`
4. login as newuser using SSH KEY:
	1.  `$ ssh newuser@PUBLIC-IP-ADDRESS -p 2200 -i ~/.ssh/id_rsa`
5. Change Password Authentication back to no:
	1. while logged as newuser, type:
	  `$ sudo nano /etc/ssh/sshd_config`
	2. change password Authentication:
	PasswordAuthentication no
	3. reload the configuration:
	  `$ sudo service ssh reload`

## 7 Configure the Uncomplicated Firewall(UFW) to only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

Source: [Udacity Configuring Linux Web Servers Course](https://www.udacity.com/course/configuring-linux-web-servers--ud299-nd)

1. Deny all incoming connections:
  `$ sudo ufw default deny incoming`
2. Allow all outgoing connections:
  `$ sudo ufw default allow outgoing`
3. Allow incoming TCP packets on port 2200 (SSH):
  `$ sudo ufw allow 2200/tcp`
4. Allow incoming TCP pakcets on port 80 (HTTP):
  `$ sudo ufw allow 80/tcp`
5. Allow incoming UDP packets on port 123 (NTP):
  `$ sudo ufw allow 123/udp`
6. enable firewall:
  `$ sudo ufw enable`
7. check status of firewall:
  `$ sudo ufw status`

## 8 Configure local timezone to UTC

Source: [Ubuntu documentation](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)

1. open timezone selection dialog
  `$ sudo dpkg-reconfigure tzdata`
2. Choose 'none of the above', then select UTC

## 9 Install and configure Apache to serve a Python mod_wsgi application

Source: [Udacity Configuring Linux Web Servers Course](https://www.udacity.com/course/configuring-linux-web-servers--ud299-nd)

1. Install Apache web server:
  `$ sudo apt-get install apache2`
2. Open a browser at your public ip address, http://54.200.104.123, and it should open up the Apache2 Ubuntu Default Page
3. Install the mod_wsgi tool:
  `$ sudo apt-get install python-setuptools libapache2-mod-wsgi`
4. Restart Apache server, so that mod_wsgi can load:
  `$ sudo service apache2 restart`

## 10 Install and configure Postgresql

1. To install PostgreSQL, type:
  `$ sudo apt-get install postgresql`
2. Check that it doesn't allow remote connections:
  `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf`

## 11 Install git, clone and setup Catalog App project

### 11.1 Install git

Source: [Github](https://help.github.com/articles/set-up-git/#platform-linux)

1. Install Git:
  `$ sudo apt-get install git`
2. Set your name:
  `$ git config --global user.name "YOUR NAME"`
3. Set your email:
  `$ git config --global user.email ""YOUR EMAIL ADDRESS"`

### 11.2 - Setup for deploying a Flask Application on Ubuntu VPS
Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps "How To Deploy a Flask Application on an Ubuntu VPS")

1. Extend Python with additional packages that enable Apache to serve Flask applications:  
  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable mod_wsgi (if not already enabled):  
  `$ sudo a2enmod wsgi`
3. Create a Flask app:  
  1. Move to the www directory:  
    `$ cd /var/www`
  2. Setup a directory for the app, e.g. catalog:  
    1. `$ sudo mkdir FlaskApp`  
    2. `$ cd FlaskApp` and `$ sudo mkdir FlaskApp`  
    3. `$ cd FlaskApp` and `$ sudo mkdir static templates`  
    4. Create the file that will contain the flask application logic:  
      `$ sudo nano __init__.py`
    5. Paste in the following code:  
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
  1. Install pip installer:  
    `$ sudo apt-get install python-pip` 
  2. Install virtualenv:  
    `$ sudo pip install virtualenv`
  3. Set virtual environment to name 'venv':  
    `$ sudo virtualenv venv`
  4. Activate the virtual environment:  
    `$ source venv/bin/activate`
  5. Install Flask inside the virtual environment:  
    `$ pip install Flask`
  6. Run the app:  
    `$ python __init__.py`
  7. Deactivate the environment:  
    `$ deactivate`
5. Configure and Enable a New Virtual Host#
  1. Create a virtual host config file  
    `$ sudo nano /etc/apache2/sites-available/FlaskApp.conf`
  2. Paste in the following lines of code and change names and addresses regarding your application:  
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
  3. Enable the virtual host:  
    `$ sudo a2ensite FlaskApp`
6. Create the .wsgi File and Restart Apache
  1. Create wsgi file:  
    `$ cd /var/www/FlaskApp` and `$ sudo nano FlaskApp.wsgi`
  2. Paste in the following lines of code:  

    ```
      #!/usr/bin/python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0,"/var/www/catalog/")
    
      from catalog import app as application
      application.secret_key = 'Add your secret key'
    ```
  3. Restart Apache:  
    `$ sudo service apache2 restart`

### 11.3 - Clone GitHub repository and make it web inaccessible
1. Clone project 3 solution repository on GitHub:  
  `$ git clone https://github.com/micahkim23/FSND-P3_Catalog-Web-App.git`
2. Move all content of created FSND-P3_Catalog-Web-App directory to `/var/www/FlaskApp/FlaskApp/`-directory and delete the leftover empty directory.
3. Make the GitHub repository inaccessible:  
  Source: [Stackoverflow](http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible)
  1. Create and open .htaccess file:  
    `$ cd /var/www/FlaskApp/` and `$ sudo nano .htaccess` 
  2. Paste in the following:  
    `RedirectMatch 404 /\.git`

### 11.4 - Install needed modules & packages
1. Install httplib2 module:
  `$ sudo pip install httplib2`
2. Install requests module:  
  `$ sudo pip install requests`
3. Install oauth2client.client:  
  `$ sudo pip install --upgrade oauth2client`
4. Install SQLAlchemy:  
  `$ sudo pip install sqlalchemy`
5. Install the Python PostgreSQL adapter psycopg:  
  `$ sudo apt-get install python-psycopg2`
6. Make sure Flask version number matches Catalog's version number :

  ```
    $ python
    >> import flask
    >> flask.__version__
  ```
7. You may or may not have to run the following code depending on your flask version:

  ```
    sudo pip install werkzeug==0.8.3
    sudo pip install flask==0.9
    sudo pip install Flask-Login==0.1.3
  ```

#### 11.5 - Run application 
1. Restart Apache:  
  `$ sudo service apache2 restart`
2. Open a browser and put in your public ip-address as url, e.g. 54.200.104.123 - if everything works, the application should come up
3. If getting an internal server error, check the Apache error files:  
  1. View the last 20 lines in the error log: 
    `$ sudo tail -20 /var/log/apache2/error.log`
  2. If a file like 'client_secrets.json' cant't been found, write full path in __init__.py:  
      1. `CLIENT_ID = json.loads(open(r'/var/www/<app_dir>/<app_dir>/client_secrets.json', 'r').read())['web']['client_id']`
      2. `oauth_flow = flow_from_clientsecrets(r'/var/www/<app_dir>/<app_dir>/client_secrets.json', scope='')`

#### 11.6 - Get OAuth-Logins Working
  Source: [Udacity](http://discussions.udacity.com/t/oauth-provider-callback-uris/20460) and [Apache](http://httpd.apache.org/docs/2.2/en/vhosts/name-based.html)  
  
1. Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 54.200.104.123, its ec2-54-200-104-103.us-west-2.compute.amazonaws.com
2. Open the Apache configuration files for the web app:
  `$ sudo nano /etc/apache2/sites-available/FlaskApp.conf`
3. Paste in the following line below ServerAdmin:  
  `ServerAlias HOSTNAME`, e.g. ec2-54-200-104-103.us-west-2.compute.amazonaws.com
4. if you go to e.g. ec2-54-200-104-103.us-west-2.compute.amazonaws.com and your application is not displayed, go to:
  `$ sudo nano /etc/apache2/sites-enabled/000-default.conf` 
  1. make sure its contents are the same as that of /etc/apache/sites-available/FlaskApp.conf
5. To get the Google+ authorization working:  
  1. Go to the project on the Developer Console: https://console.developers.google.com/project
  2. Navigate to APIs & auth > Credentials > Edit Settings
  3. add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-54-200-104-103.us-west-2.compute.amazonaws.com/oauth2callback
6. This would have changed your client_secrets.json file, so grab the new file and update your old file.
7. Restart Apache:
  `$ sudo service apache2 restart`

