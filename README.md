# Linux Server Configuration

user: grader

IP Address: 34.199.16.1

SSH Port: 2200

Url: [34.199.16.1](http://34.199.16.1)

## Configuration Details

### SSH into Server

1. Download the private key


2. Login into server as root using the command:

```
ssh root@34.199.16.10 -i ~/.ssh/insertprivatekey.pem
```

### Update all existing software

Run the following commands:

```
sudo apt-get update
```

Then:

```
sudo apt-get upgrade
```

### Create User grader

1. Use the command below to create a new user:

```
sudo adduser grader
```
2. Give the user grader sudo permission by:

```
sudo nano /etc/sudoers.d/grader
```

In the file add the following text and save with `CTRL + O` and exit with `Ctrl + X`:

```
grader ALL=(ALL:ALL) ALL
```
### Configure localtime

1. Configure the time using `sudo dpkg-reconfigure tzdata`

### Configure Key Based Authentication

1. On your local machine generate a ssh key pair with the command

```
ssh-keygen
````

2. Switch users with the command `su grader` and enter the password you gave

3. Change to your home directory with:

```
cd ~
```

4. Now create file for authorized keys with:

```
sudo touch .ssh/authorized_keys
```
Then type in:

```
sudo nano .ssh/authorized_keys
```

5. On your local machine use the command:

```
sudo cat publickey.pub
```

6. Copy the contents and paste them into the authorized keys file on the server and save.

7. Change the file permissions for the authorized key files with the commands below:

```
chmod 700 .ssh

chmod 644 .ssh/authorized_keys
```

### Forcing Key Based Authentication, Disable Root and change SSH port


1. Run the command `sudo nano /etc/ssh/sshd_config`

2. On the top there should be a port line change it to **Port 2200**

3. Find the line **PasswordAuthentication** and change it to **no**

4. Find the line **PermitRootLogin** and change it to **no**

5. Save the file and restart the ssh service with `sudo service ssh restart`

### Configure Uncomplicated Firewall

Run the commands to set ports and enable the firewall:

```
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/tcp
sudo ufw enable
```
### Install Necessary Software for Hosting

Run the following commands:

```
sudo apt-get install git
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
sudo apt-get install postgresql
sudo apt-get install python-pip
sudo pip install sqlalchemy
sudo pip install psycopg2
sudo pip install httplib2
sudo pip install requests
sudo pip install flask
```
### Configure PostgreSQL

1. Switch to user postgres with `sudo su postgres`

2. Run `psql` to get into postgresql shell

3. Create database using the command:

```
CREATE DATABASE catalog;
```

4. Create user catalog and change the role with the following commands:

```
CREATE User catalog;
ALTER ROLE catalog WITH PASSWORD 'password';
```

5. Transfer database privileges to catalog user with:

```
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```

6. Quit psql shell with `\q` and `exit` to switch back to grader

### Setting up the project

1. Change the directory with `cd /var/www/`

2. Create a directory with `sudo mkdir FlaskApp`

3. Run the command to get the project:

```
git clone https://github.com/arthurchan1111/catalog.git

```

4. Rename the project using:

```
sudo mv ./catalog ./FlaskApp
```

5. Move into the project directory and rename using the following command

```
sudo mv application.py __init__.py
```

6. Using the `sudo nano` command change the following line in `__init__.py`and `database_setup.py` from:

```
engine = create_engine('postgresql:///catalog')
```

To:

```
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```

7. Create the database schema with:

```
sudo python database_setup.py
```

### Configure Hosting

1. Use the command `sudo nano /etc/apache2/sites-available/FlaskApp.conf` and copy the contents below:

```
<VirtualHost *:80>
        ServerName 34.199.16.1
        ServerAdmin arthurchan1111@gmail.com
        WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/FlaskApp/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/FlaskApp/FlaskApp/static
        <Directory /var/www/FlaskApp/FlaskApp/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Save the file then exit.

2. Use the command `sudo nano /var/www/FlaskApp/flaskapp.wsgi` and copy the contents below:

```
#!/usr/bin/env python

import sys
import logging
logging.basicConfig(stream=sys.stderr)

sys.path.insert(0, '/var/www/FlaskApp/')

from FlaskApp import app as application
application.secret_key= 'super_secret_key'
```

Save the file then exit.

3. Restart the service with `sudo service apache2 restart`

### References

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#step-five-%E2%80%93-create-the-wsgi-file
