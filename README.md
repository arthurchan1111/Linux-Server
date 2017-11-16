# Linux Server Configuration

user: grader

IP Address: 34.224.56.60

SSH Port: 2200

Url: [34.224.56.60](34.224.56.60)

## Configuration Details

### SSH into Server

1. Download the private key


2. Login into server as root using the command:

```
ssh root@34.224.56.60 -i ~/.ssh/insertprivatekey.pem
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

4. 
