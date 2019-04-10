# Project: Linux Server Configuration
To complete this Project I have used Amazon Lightsail for a Ubuntu Linux server instanse. To secure my server I have used Uncomplicated Firewall(UFW) to only allow incoming connections for SSH(port 2200), HTTP(port 80),and NTP(port 123).
## Getting Started

### IP address
This static IP is available for public connection worldwide.
18.185.212.159 
ssh port: 2200
### URL
You can visit my catalog item project [here](http://18.185.212.159:80/)
### summary of software installed
* Apache
To get my server responding to HTTP requests i have installed Apache HTTP Server.
* mod_wsgi
Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications.
* PostgreSQL
A database server to store my application's data persistent.
* Flask
create a virtual environment for my flask application. use pip to install virtualenv and Flask
* Psycopg2
Psycopg is the most popular PostgreSQL adapter for the Python programming language.
### summary of configurations made
#### Firewall configurations 

1. Setting up Default Policies

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

These commands set the defaults to deny incoming and allow outgoing connections. 

2. Allowing SSH Connections

By default is the port 22 that the SSH daemon listens on.
I have used the command below to let my SSH daemon listen on a different port .

```
sudo ufw allow 2200
```

3. Allowing Other Connections

HTTP on port 80, which is what unencrypted web servers use, using sudo ufw allow http or sudo ufw allow 80
NTP(The Network Time Protocol) on port 123,which is a networking protocol for clock synchronization between computer systems over packet-switched, variable-latency data networks. I have used sudo ufw allow ntp.

4. Enabling UFW

To enable UFW, use this command:
```
sudo ufw enable
```

#### create new user and give the user Sudo Access

1. Use the adduser command to add a new user to my system.
2. Add the new user to the sudoers file 
I have used the visudo command:
```
sudo visudo
```
then add the following line after the comment line, “User privilege specification”:
```
xxx  ALL=(ALL:ALL) ALL
```
Then,save the file.
3. Install the public key for the new user
We cannot log into the new user account via SSH until the public key from the linux instance’s key pair is installed for the new user. We must copy the public key installed for the ubuntu user and paste it into the right file in the new user account. The public key is in the file, ~/.ssh/authorized_keys.
Now, switch to the new user account,ensure you are in the new user's home directory,then create the SSH directory and authorized users file,with the correct permissions:
```
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 644 .ssh/authorized_keys
```
Edit the authorized_keys file with a text editor.Paste in the public key you previously copied to the clipboard.Save the file.
#### Configure the local timezone to UTC
simply execute
```
sudo dpkg-reconfigure tzdata
```
then scroll to the bottom of the Continents list and select None of the above;
in the second list, select UTC.
#### configure Apache
Issue the following command in terminal:
```
sudo nano /etc/apache2/sites-available/FlaskApp.conf
```
Add the following lines of code to the file 
```
<VirtualHost *:80>
                #ServerName mywebsite.com
                ServerAdmin admin@mywebsite.com
                WSGIScriptAlias / /var/www/Item_Catalog/vagrant/item_catalog.wsgi
                <Directory /var/www/Item_Catalog/vagrant/catalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/Item_Catalog/vagrant/catalog/static
                <Directory /var/www/Item_Catalog/vagrant/catalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /templates /var/www/Item_Catalog/vagrant/catalog/templates
                <Directory /var/www/Item_Catalog/vagrant/catalog/templates/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                 LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
Save and close the file.
Enable the virtual host with the following command:
```
sudo a2ensite FlaskApp
```
#### Create the .wsgi File under your applications directory
 Move to the /var/www/Item_Catalog/vagrant/ directory and create a file named item_catalog.wsgi with following commands:
 ```
    sudo nano item_catalog.wsgi 
 ```
 Add the following lines of code to the item_catalog.wsgi file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Item_Catalog/vagrant/catalog")

from application import app as application
application.secret_key = 'super_secret_key'

```
Save and close the file.
#### configure PostgreSQL
1. forbid remote connections
We can double check that no remote connections are allowed by looking in the host based authentication file:
```
sudo nano /etc/postgresql/9.1/main/pg_hba.conf
```
2. create a new database user that has limited permissions to the database.
As the default configuration of Postgres is, a user called postgres is made on and the user postgres has full superadmin access to entire PostgreSQL instance running on OS.
```
sudo -u postgres psql
```
This command gets you the psql command line interface in full admin mode.
then simple run:
```
CREATE ROLE xxx WITH ENCRYPTED PASSWORD 'xxx' CREATEDB LOGIN;
```
3. create a new database
```
CREATE DATABASE xxx OWNER xxx
```
#### changes in my Item Catalog project
1. for my database set up I have used sqlalchemy,  I need a data engine for sqlalchemy, before I have used sqlite for the engine,now I have to change it to postgresql+psycopg2.
```
engine = create_engine(
    'postgresql+psycopg2://username:userpasswords@localhost:5432/database'
)

```

Finally, restart Apache with the 
```
sudo apache2ctl restart
```
 command.
### list of third-party resources
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#step-four-%E2%80%93-configure-and-enable-a-new-virtual-host)
* [How to set up a new user on your Amazon AWS server](https://www.brianlinkletter.com/how-to-set-up-a-new-userid-on-your-amazon-aws-server-instance/)
* [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
* [How To Set Up a Firewall with UFW on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-16-04#step-4-%E2%80%94-enabling-ufw)
* [postgresql documentation](https://www.postgresql.org/docs/9.0/)
