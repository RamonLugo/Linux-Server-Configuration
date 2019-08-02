#Linux Server Configuration Project

This project is a baseline installation of a Linux distribution on a virtual machine.  It is prepared to host web applications.  It includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

This is running on AWS Lightsail.  It is a project within the Udacity Full Stack Web Developer Nano Degree.


##Additional Information
It also allows for password-less logon using a generated private key on the server and a public key on the client.

The server will be used to serve the Catalog website that was another project in this course.

## Server Details
IP address: 3.80.23.136 
SSH Port: 2200  
WWW Port: 80  
NTP Port: 123  
Username: grader  
URL site: <http://3.80.23.136/>  or <http://ec2-3-80-23-136.compute-1.amazonaws.com>  
**Reference**
(https://serverfault.com/questions/403440/print-external-host-name-of-ec2-instance)  
use command `curl http://instance-data.ec2.internal/latest/meta-data/public-hostname` to get instance name.
No need for password Logon:  
ssh -i ~/.ssh/LightsailDefaultKey.pem -p 2200 grader@3.80.23.136. 

## Installed Packages
Name | Description
-----|------------
apache2 | Web Server
libapache2-mod-wsgi | Tool that serves Python web apps from Apache server
postgresql | Relational database system
sqlalchemy | SQL toolkit and ORM for Python
Flask | Python web framework
oauth2client | Python library for accessing resources protected by OAuth 2.0
httplib2 | A comprehensive HTTP client library
requests | Python HTTP Requests for Humans
git | Version control tool  

##Create a AWS Lightsail Instance 
1. Get an account on Amazon AWS
2. Create a Lightsail instance
3. Choose the Ubuntu OS
4. OS is Ubuntu 18.04.2 LTS 
5. The new instance's security group has the SSH port 22 by default
6. The public IP is 3.80.23.136
7. Download the private key LightsailDefaultKey-us-east-1.pem from your account page, get it from the section titles "Manage your SSH keys"  
8. Rename the key **LightsailDefaultKey.pem** amd move it to your `~/.ssh/` directory.  

##Update all currently installed packages 
1. $ `sudo apt-get update`
2. $ `sudo apt-get upgrade`  

##User Configuration
1. Log into the remote VM as root user (ubuntu) through ssh: `$ ssh -i ~/.ssh/LightsailDefaultKey.pem -p 2200 ubuntu@18.212.208.248
2. Create a new user grader: `$ sudo adduser grader`
3. Grant grader the permission to sudo, by adding a new file under the suoders directory: $ sudo nano/etc/sudoers.d/grader . In the file put in: grader ALL=(ALL:ALL) ALL , then save and quit  
4. Create a new user grader: `$ sudo adduser catalog`
5. Grant grader the permission to sudo, by adding a new file under the suoders directory: $ sudo nano/etc/sudoers.d/catalog . In the file put in: catalog ALL=(ALL:ALL) ALL , then save and quit  
6. Check the new users are present as root:  `$ cut -d: -f1 /etc/passwd`  
  
##SSH and Security Configurations
1. Generate a new key pair by entering the following command at the terminal of your local machine.`$ ssh-keygen ` with id_grader_key. 
2. Print the public key `$ sudo cat ~/.ssh/id_grader_key.pub`  
3. Select the public key and copy it.  
4. Create a new directory called .ssh `$ sudo mkdir /home/grader/.ssh` on your virtual machine. 
5. Create a new file call authorized_keys `$ sudo nano /home/grader/.ssh/authorised_keys ` on your virtual machine. 
5. Paste the public key grader_key.pub to authorized_keys , and change the permissions:  
	1. `$ sudo chmod 700 /home/grader/.ssh`. 
	2. `$ sudo chmod 600 /home/grader/.ssh/authorized_keys`.   
	3. Change the owner from ubuntu to grader : `$ sudo chown -R grader:grader /home/grader/.ssh`. 6. Enforce key-based authentication, change SSH port to 2200 and disable remote login of root user:
	1. `$ sudo nano /etc/ssh/sshd_config`
	2. Change Port to 2200
	3. Change PermitRootLogin to no  	5. Restart the ssh service - `$ sudo service ssh restart` 

##Configure the local timezone to UTC
1. Open time configuration and set it to UTC: `$ sudo dpkg-reconfigure tzdata`
2. Install ntp daemon ntpd for a better synchronization of the server's time over the network connection: `$ sudo apt-get install ntp`  
**References**
Ubuntu Wiki, [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)  
Ask Ubuntu, [How do I change my timezone to UTC/GMT?](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)  

##Configure the Uncomplicated Firewall (UFW)
1. Project requires that the server only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)  
2. `$ sudo ufw default deny incoming`  
3. `$ sudo ufw default allow outgoing` 
4. `$ sudo ufw allow 2200/tcp`  
5. `$ sudo ufw allow 80/tcp`  
6. `$ sudo ufw allow 123/udp`  
7. `$ sudo ufw enable` 
8. Check the status of UFW to list current roles: `sudo ufw status`
9. Add to lightsail management console: Application = Custom, Protocol = TCP, Port = 2200 (https://askubuntu.com/questions/1019891/connecting-to-amazon-lightsail-ubuntu-server-using-different-ssh-port)  
10. Click on the `Manage` option of the Amazon Lightsail Instance, then the `Networking` tab, and then change the firewall configuration to match the internal firewall settings.  
11. Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22 by deleting it in the console.  
12. Log into the remote VM as grader through ssh: `$ ssh -i ~/.ssh /id_grader_key -p 2200 grader@118.212.208.248`.
**References**  
Official Ubuntu Documentation, [UFW - Uncomplicated Firewall](https://help.ubuntu.com/community/UFW).  
TechRepublic, [How to install and use Uncomplicated Firewall in Ubuntu](https://www.techrepublic.com/article/how-to-install-and-use-uncomplicated-firewall-in-ubuntu/).  

##Installing pip
Install latest version of pip for Python 3 `$ sudo apt install python3-pip`  
To confirm whether or not it has been successfully installed, run: `$ pip3 --version`

##Install Apache, mod_wsgi and Git
1. Install apache2 `$ sudo apt-get install apache2 `
2. Install the Python 3 mod_wsgi package: $ `sudo apt-get install libapache2-mod-wsgi python3-dev`
3. Enable mod_wsgi: ` $ sudo a2enmod wsgi` 
4. Restart apache2 `$ sudo service apache2 start` 
5. Install git `$ sudo apt-get install git` 
6. Configure git:  
	1. `$ git config --global user.name "your full name"`
	2. `$ git config --global user.email "myemail@domain.com"`  

##Apache Fix
To selectively block public access to all files under the .git directory.  
`sudo nano /etc/apache2/conf-enabled/security.conf`  
Add `<DirectoryMatch "/\.git">`  
		`Require all denied`  
	`</DirectoryMatch>`   
(https://davidegan.me/hide-git-repos-on-public-sites/)  

If you DO NOT have access to Apache config files, add these lines to a .htaccess file in your project root:
Make .git file inaccessable
`$ sudo nano .htaccess`
add ` RedirectMatch 404 /\.git `

##Install and configure PostgreSQL
Install PostgreSQL with:  
`sudo apt-get install postgresql postgresql-contrib`  
To ensure that remote connections to PostgreSQL are not allowed, check that the configuration file `/etc/postgresql/10/main/pg_hba.conf` only allowed connections from the local host addresses 127.0.0.1 for IPv4 and ::1 for IPv6  
you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```

Default postgres super user and postgres database is automatically created. Set a password for the postgres super user  
`$ sudo passwd postgres` 
Switch to the `postgres` user: `sudo su - postgres`  
Open PostgreSQL interactive terminal with `psql   
Set the password. `# \password postgres`  
You are prompted for a password.  
**Reference**   
(https://serverfault.com/questions/110154/whats-the-default-superuser-username-password-for-postgres-after-a-new-install)

Create the `catalog` user with a password and give them the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```

- List the existing roles: `\du`. The output should be like this:
  
                                    List of roles  
   Role name |                         Attributes                         | Member of     
  -----------|------------------------------------------------------------|-----------
   catalog   | Create DB                                                  | {}
   postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
  
   
- Create the 'catalog' database owned by catalog: `# CREATE DATABASE catalog WITH OWNER catalog;`  
- Connect to the database: `# \c catalog `  
- Revoke all the rights: `# REVOKE ALL ON SCHEMA public FROM public;`  
- Lock down the permissions to only let catalog role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`  

- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`
Check the status:  
`$ service postgresql status`  
- Log in as user catalog: `su - catalog`, enter the password  
- Run `psql` and then run `\l` to see that the new database has been created. The output should be like this:
  ```
                                    List of databases
     Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
  -----------+----------+----------+-------------+-------------+-----------------------
   catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
   postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
   template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
   template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
  (4 rows)
  ```
- Exit psql: `\q`.
- Log out from user `catalog` ahd return to the `grader` user: `$ exit`  
**Reference**   
(https://askubuntu.com/questions/831292/how-do-i-install-postgresql-9-6-on-any-ubuntu-version)  
 
##Install packages: 
Update: `$ sudo apt-get update`   
Install Python's PostgreSQL adapter psycopg2: `$ sudo apt-get install python3-psycopg2 python3-flask`  
Install Python's sqlalchemy: `$ sudo apt-get install python3-sqlalchemy ` 

##Configure Apache to serve a Python mod_wsgi application
###Clone the item-catalog app from Github  
Change the current working directory to /var/www/:  
1. `$ cd /var/www `   
2. `$ sudo git clone https://github.com/RamonLugo/catalog.git catalog`  
3. `$ sudo chown -R grader:grader catalog `   

Change to the ` cd /var/www/catalog/catalog` directory:
1. Edit the application.py, createDataForDatabase.py and database_setup.py file:  
2. Change `engine = create_engine('sqlite:///category.db')` to  
 `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
3. Change file application.py to init.py: `$ mv application.py __init__.py`  
4. Edit the __init__.py file nano `__init__.py`
5. Change line `app.run(host='0.0.0.0', port=8000)` to `app.run()`  
6. Install other packages needed for the Catalog project  
1. `$ sudo pip3 install -r requirements.txt`  

##Modify Google Credentials
Logon to [Google Cloud Plateform](https://console.cloud.google.com/)
Go to "Credentials"  
Edit the Authorized domains: add nip.io.  
Edit the OAuth Client - add http://3.80.23.136.nip.io as authorized JavaScript origins.  
Add http://3.80.23.136.nip.io/login and http://3.80.23.136.nip.io/gconnect to the Authorized redirect URIs section.
Download the JSON file, open it and copy the contents.  
Replace the text in  `/var/www/catalog/catalog/client_secrets.json`.

##Install the Virtual Environment
Install the virtual environment: `sudo apt-get install python-virtualenv`  
Change to the catalog directory: `cd /var/www/catalog/catalog/`  
Create the virtual environment: `sudo virtualenv -p python3 venv3`.
Make the user grader the owner: `sudo chown -R grader:grader venv3/`.  
Activate the environment: `. venv3/bin/activate`.

Install the following dependencies:
  ```
  pip3 install httplib2
  pip3 install psycopg2
  pip3 install sqlalchemy
  pip3 install requests
  pip3 install --upgrade oauth2client   
  pip3 install flask
  sudo apt-get install libpq-dev    
  ```  
  Deactivate the virtual environment: `deactivate`.  

##Setting Up the Virtual Host  
Edit the wsgi.conf file to use Python 3: `sudo nano /etc/apache2/mods-enabled/wsgi.conf`.

  ```
  #WSGIPythonPath directory|directory-1:directory-2:...
  WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.6/site-packages
  ```

- Run the following command in terminal to set up a file called catalog.conf to configure the virtual hosts: `sudo nano /etc/apache2/sites-available/catalog.conf` and add the following lines to configure the virtual host:

  ```
  <VirtualHost *:80>
      ServerName 3.80.23.136
      ServerAlias ec2-3.80.23.136.compute-1.amazonaws.com
      ServerAdmin admin@3.80.23.136
      WSGIDaemonProcess catalog user=www-data group=www-data threads=5
      WSGIProcessGroup catalog
      WSGIApplicationGroup %(GLOBAL)
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi
      <Directory /var/www/catalog/catalog/>
          Require all granted
      </Directory>
      Alias /static /var/www/catalog/catalog/static
      <Directory /var/www/catalog/catalog/static/>
          Require all granted
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
  ```
Require Directive
**Resource**   
[https://stackoverflow.com/questions/10873295/error-message-forbidden-you-dont-have-permission-to-access-on-this-server]  

- Enable virtual host: `sudo a2ensite catalog`.  
- The will see:
  ```
  Enabling site catalog.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

- Then reload Apache: `sudo service apache2 reload`.

**Resources** 
- [Getting Flask to use Python3 (Apache/mod_wsgi)](https://stackoverflow.com/questions/30642894/getting-flask-to-use-python3-apache-mod-wsgi)
- [Run mod_wsgi with virtualenv or Python with version different that system default](https://stackoverflow.com/questions/27450998/run-mod-wsgi-with-virtualenv-or-python-with-version-different-that-system-defaul)

##Set up the Flask application  
Create the catalog.wsgi file `sudo nano /var/www/catalog/catalog.wsgi` and add the following lines:

  ```
  activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
  with open(activate_this) as file_:
      exec(file_.read(), dict(__file__=activate_this))

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/catalog/")
  sys.path.insert(1, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = "..."
  ```

- Then restart Apache: `sudo service apache2 restart`.

**Resource** 
- Flask documentation, [Working with Virtual Environments](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#working-with-virtual-environments)

##Populate the database
- Chnage the directory  `cd /var/www/catalog/catalog/` directory,  
- Edit `/var/www/catalog/catalog/database_setup.py`.
- Add the these two lines at the beginning of the file.  
  ```
  import sys
  sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python3.6/site-packages") 
  ```  
  
- Edit `/var/www/catalog/catalog/createDataForDatabase.py `.  
- Add the these two lines at the beginning of the file.  
  ```
  import sys
  sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python3.6/site-packages") 
  ```  
  
- Edit `/var/www/catalog/catalog/__init__.py `.
- Add the these two lines at the beginning of the file.  
  ```
  import sys
  sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python3.6/site-packages") 
  ```
- Edit: `sudo nano /var/www/catalog/catalog/__init__.py`  
- Change line `open('client_secrets.json', 'r').read())['web']['client_id']` to   
     `open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']`  
- activate the virtual environment: `. venv3/bin/activate`.
- Create the database: `python3 database_setup.py`.  
- Create data in the databases: `python3 createDataForDatabase.py`.
- Deactivate the virtual environment: `deactivate`.

##Disable the default Apache site  
- Disable the default Apache site: `sudo a2dissite 000-default.conf`.  
The following prompt will be returned:  
  `Site 000-default disabled.`  
 ` To activate the new configuration, you need to run:`  
    `service apache2 reload`
- Reload Apache: `sudo service apache2 reload`.  

##Install Sendmail for email support
- Enter: `sudo apt-get install sendmail`.  

##Launch the Web Application 
- Change the directory `cd ..` 
- Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
- Restart Apache again: `sudo service apache2 restart`.
- Enter `python3 __init__.py`.  
- Open your browser to http://3.80.23.136 or http://ec2-3.80.23.136.compute-1.amazonaws.com.
- To Logon: ttp://3.80.23.136/login or http://ec2-3.80.23.136.compute-1.amazonaws.com/login

##Resources
Lot of help from   
`https://knowledge.udacity.com/?page=1&query=ggoogle%20oauth%20does%20not%20redirect&sort=RELEVANCE` especially on Google oauth.   
  
From various repos:  
[anumsh/Linux-Server-Configuration](https://github.com/anumsh/Linux-Server-Configuration)  
[boisalai/udacity-linux-server-configuration](https://github.com/boisalai/udacity-linux-server-configuration)  

WebSites.  
[Ubuntu](https://www.ubuntu.com).   
[DigitalOcean](https://www.digitalocean.com)

##Reviewer please read this 
[Udacity Knowledge Post](https://knowledge.udacity.com/questions/22059).  
I have been having issues for over a month and finally realized that it wasn't me.








