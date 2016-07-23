# Linux Configuration  
Linux Server Configuration - Project 5 (Final) of Udacity's Full Stack Nanodegree Program  

* Website: http://ec2-52-25-79-87.us-west-2.compute.amazonaws.com/  
* IP Address: 52.25.79.87  
* SSH port: 2200  

Login with command `ssh -p 2200 -i ~/.ssh/udacity_key.rsa root@52.25.79.87`

###Software Installed  
* python-pip  
* apache2  
* libapache2-mod-python  
* libapache2-mod-wsgi  
* sqlalchemy  
* postgresql  
* postgresql-contrib  
* virtualenv (but I ended up not using it - can't wrap my head around it)  
* flask  
* git  

###Server Configurations Made  
* Change SSH from 22 to 2200  in /etc/ssh/sshd_config  
* Restarted SSH service to reset this port  
* Configure UFW to  
  * `default deny incoming`  
  * `default allow outgoing`  
  * `allow ssh` (2200)  
  * `allow www` (80)  
  * `allow ntp` (123)  
* Make user grader, and give permission to sudo via sudoer.d  
* Configure local timezone to UTC via `dpkg-reconfigure tzdata`  

##PostgreSQL Configurations Made  
* Switch to the postgres user via `su - postgres`  
* Create an owner for the database to be used with postgres command `create user relaymaster with password 'mypassword';`  
* Create database instance for relay library with postgres command `create database relaydatabase owner relaymaster encoding 'utf-8';`  
* Logged into the relaydatabase to create and change user permissions:  
  * `createuser catalog` to make a user  
  * `REVOKE ALL ON SCHEMA public FROM catalog` to limit permissions (not sure if this does the trick)  
  * `GRANT ALL ON SCHEMA public TO relaymaster` to ensure proper permissions  

###WebApp Configurations Made  
* `git clone` item catalog project into folder /var/www/FlaskApp/  
* Create WSGI structure in /var/www/FlaskApp/ with following code:  
```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/itemcatalog")

from project import app as application
application.secret_key = 'Add your secret key'
```  

* Give item catalog webapp an apache enabled structure though creating FlaskApp.conf in /etc/apache2/sites-available containing the following code:  
```apacheconf
<VirtualHost *:80>
                ServerName ec2-52-25-79-87.us-west-2.compute.amazonaws.com
                ServerAdmin tfudacity@gmail.com
                WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
                <Directory /var/www/FlaskApp/itemcatalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/FlaskApp/itemcatalog/static
                <Directory /var/www/FlaskApp/itemcatalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```  
...then activating through command `a2ensite FlaskApp`  
* Recreate app on Google and Facebook to gain new OAuth2 secrets and login capability  
* Recreate `client_secrets.json` and `fb_client_secrets.json`  
* Change `python.py` to absolutely reference `client_secrets.json`, `fb_clientsecrets.json` where necessary  
* Change /templates/login.html to reference the new client secrets  
* Delete downloaded relaydatabase file  
* Change `database_setup.py`, `relaypopulator.py` and `project.py` to reference the newly made PostgreSQL server made, `relaydatabase`, via the line:  
```python
engine = create_engine('postgresql://relaymaster:mypassword@localhost/relaydatabase')
```  
* Reran `database_setup.py` and `relaypopulator.py`  
* Restart apache with `apache2ctl restart`  
* Tested to make sure nothing broke by going to FlaskApp folder and running `flaskapp.wsgi` via `python`  

###The End(?) 
