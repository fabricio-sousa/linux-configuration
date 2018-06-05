# Udacity Full Stack Web Developer Nanodegree | Linux Server Configuration Project

### Developer: Fabricio Sousa

## Description

From Udacity's course description: "You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers."

## Server Info

* IP address: 18.221.122.149
* SSH Port: 2200
* App URL: http://ec2-18-221-122-149.us-east-2.compute.amazonaws.com/

## Frameworks, Libraries, and APIs

HTML5 and CSS, Python 2.7, Javascript, PostgreSQL, Flask.

## Software Used For This Project

* `fail2ban` - to help prevent brute-force server attacks.
* `sendmail` - to send the admin mail anytime suspicious activity is detected via fail2ban.
* `unattended-upgrades` - to allow automatic system updates.
* `apache2` - installs Apache Web Server package.
* `libapache2-mod-wsgi python-dev` - enables Apache to serve Flask apps
* `git` - for version control and cloning the remote repository.
* `pip` - allows for Python package installs.
* `virtualvenv` - installs the virtual environment.
* `Flask` - to install the Flask module.
* Project dependencies software to allow Python to interact with Flask, the webserver and database:
`bleach`, `httplib2`, `request`, `oauth2client`, `sqlalchemy`, `python-psycopg2`
* `libpq-dev python-dev` and `postgresql postgresql-contrib` - installs Python packages for PostgreSQL.


## Installation and How To Run (Linux Server Configuration and app deployment)

### Get your server.

1. Start a new Ubuntu Linux server instance on Amazon Lightsail
* Access [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) and follow instructions to create an account. 
* Click `Create Instance`
* Choose the Linux platform, OS Only, and Ubuntu.

2. SSH into the server

* Once initialized, click the instance you just created, click "Account" at the bottom, and download the private key.
* Move the downloaded key into your local computer folder `~/.ssh` and rename it to `ls_key.rsa`
* In Git Bash, type `chmod 600 ~/.ssh/ls_key.rsa`
* Use the following command to connect to the instance via Git Bash:
`ssh -i ~/.ssh/ls_key.rsa ubuntu@18.221.122.149`

### Secure your server.

1. Update and upgrade installed packages: 
* `sudo apt-get update`
* `sudo apt-get upgrade`

2. Set the SSH port from 22 to 2200:

* First go to the Instances main page, click `Networking` and click `Add Another` two times. Then add `Custom UDP` with value of 123; then `Custom TCP` with value of 2200. Click Save.
* Next, in Git Bash, type: `sudo nano /etc/ssh/sshd_config` and change the port value from 22 to 2200.
* Save and exit the nano editor.
* Restart the SSH via `sudo service ssh restart`

3. Configure the UFW Firewall:

* We are configuring the Ubuntu firewall to allow connections from port 2200 (SSH), 80 (HTTP) and 123 (NTP), as follows:

```
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw deny 22
sudo ufw enable
```
* Exit the connection by typing `exit` then `enter`.
* Return to the instance firewall management page in Amazon Lightsail and delete port 22.
* `ssh -i ~/.ssh/ls_key.rsa ubuntu@18.221.122.149`

4. Install fail2ban to help prevent brute-force server attacks:
* Install the package via `sudo apt-get install fail2ban`
* Install the sendmail package via`sudo apt-get install sendmail iptables-persistent`
* Generate a file to customize fail2ban via `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
* `sudo nano /etc/fail2ban/jail.local` and change the settings as follows:

```
destemail = your email
set bantime = 600
action = %(action_mwl)s
```

* Still in the same edit screen, under `[sshd]` change the port ssh to 2200.
* Save and exit the nano editor.
* `sudo service fail2ban restart`
* Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)

5. Allow automatic system updates:
* `sudo apt-get install unattended-upgrades`
* `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades` and uncomment the line: `${distro_id}:${distro_codename}-updates`
* Save and close the nano editor.
* `sudo nano /etc/apt/apt.conf.d/20auto-upgrades` and edit as follows:

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

* `sudo dpkg-reconfigure --priority=low unattended-upgrades` Initializes it.

6. Disable root login:
* `sudo nano /etc/ssh/sshd_config` and set `PermitRootLogin` to no.
* `sudo service ssh restart`

7. Update all packages
`sudo apt-get update`
`sudo apt-get dist-upgrade`
`sudo shutdown -r now`
* `ssh -i ~/.ssh/ls_key.rsa ubuntu@18.221.122.149` Back in.
* Source: [Digital Ocean](https://www.digitalocean.com/community/questions/updating-ubuntu-14-04-security-updates)

### Give grader access.

1. Create user account `grader`:
* While still as ubuntu user, run `sudo adduser grader`
* Enter password twice and complete creation.
* Give grader sudo access via: `sudo nano /etc/sudoers.d/grader` and add the line, `grader ALL=(ALL:ALL) ALL`
* Save and exit the nano editor.
* Configure time zone to UTC via `sudo dpkg-reconfigure tzdata`

2. Grant a SSH key pair for grader:
* Open a new Git Bash terminal but do not SSH (local machine).
* `ssh-keygen -f ~/.ssh/grader_key` where `grader_key` is your chosen key name.
* `cat ~/.ssh/grader_key.pub` and copy the key contents. Warning: Copy the contents as is, one giant line with no breaks.
* Return to the grader's ssh terminal.
* Create a new dir via `mkdir .ssh`
* `sudo chown -R grader:grader /home/grader/.ssh` to grant grader permission to that dir.
* `sudo chown -R grader:grader /home/grader/.ssh/authorized_keys` to grant grader permission to that file.
* `chmod 700 .ssh` to grant file permissions.
* `chmod 644 .ssh/authorized_keys` to grant file permissions.
* `sudo nano /etc/ssh/sshd_config` and make sure `PasswordAuthentication` is set to no.
* `sudo service ssh restart`
* `exit` and then `ssh -i ~/.ssh/grader_key -p 2200 grader@18.221.122.149`
* Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)

### Prepare the Server and Deploy your App!

1. Install Apache:

* As grader, install Apache via `sudo apt-get install apache2`
* Enable Apache to serve Flask apps via `sudo apt-get install libapache2-mod-wsgi python-dev`
* Enable the Apache HTTP server module WSGI via `sudo a2enmod wsgi`
* Start it via `sudo service apache2 start`
* If you load up your public IP in your browser, you should see a "It works`" page.

2. Install Git:

* `sudo apt-get install git`

3. Clone the Catalog app from your bithub repo:

* `cd /var/www`
* `sudo mkdir catalog` to create the catalog dir.
* `cd /catalog` then `git clone https://github.com/fabricio-sousa/items-catalog.git catalog`
* Create a catalog.wsgi file so that it can run the web app over the apache mod_wsgi, as follows:
`sudo nano var/www/catalog/catalog.wsgi` and add:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "super_secret_key"
```

* `sudo service apache2 restart`
* Source: [mod_wsgi](http://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIScriptAlias.html)

4. Update the OAuth client IDs and Javascript origins

* Go to https://console.cloud.google.com
* Create a new OAuth Client ID using http://18.221.122.149 and http://ec2-18-221-122-149.us-east-2.compute.amazonaws.com as Authorized Javascript Origins
* Update the catalog app's `login.html` with the new Client ID.
* Download the JSON file from Google, copy the contents and edit the previous `client_secrets.json` file in the catalog folder and replace its contents with the new JSON file contents you just downloaded.

5. Install pip, virtualenv (Virtual Environment), flask and dependencies:

* `sudo apt-get install python-pip` This installs pip which is used to install Python packages.
* `sudo pip install virtualenv` This installs the virtual environment.
* `cd /var/www/catalog` then `sudo virtual env venv` to start a new virtual env.
* Activate venv via `source venv/bin/activate`
* Grant permissions via `sudo chmod -R 777 venv`
* `pip install Flask`
* Install the remaining dependencies for the web app:
`pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2`
* Sources: [Flask](http://flask.pocoo.org/docs/0.12/installation/), [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps_)

6. Configure the Virtual Host

* `sudo nano /etc/apache2/sites-available/catalog.conf` to config a new virtual host.
* Add the following to the new file:
```
<VirtualHost *:80>
    ServerName 18.221.122.149
    ServerAlias ec2-18-221-122-149.us-east-2.compute.amazonaws.com
    ServerAdmin admin@18.221.122.149
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2$
    WSGIProcessGroup catalog
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
* Enable the host via `sudo a2ensite catalog`
* Source: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)

7. Configure PostgreSQL and modify the app to use it

* `sudo apt-get install libpq-dev python-dev` Installs Python packages for PostgreSQL
* `sudo apt-get install postgresql postgresql-contrib`
* Change to the new automatically created postgres user `sudo su - postgres`
* Connect to the database environment via `psql`
* Create a new `catalog` user via: `# CREATE USER catalog WITH PASSWORD 'catalog';
* `ALTER USER catalog CREATEDB;` Gives catalog user the capability to create a database.
* `CREATE DATABASE catalog WITH OWNER catalog;`
* `\c catalog` to connect to the newly created database
* `REVOKE ALL ON SCHEMA public FROM public;` The catalog user must have limited permissions.
* `GRANT ALL ON SCHEMA public TO catalog;`
* `\q` then `exit` to return to user grader.
* Rename the `app.py` file via `mv application.py __init__.py`
* Modify all entries with `engine = create_engine("sqlite:///catalog.db")` in the `__init__.py` and `database_setup.py` and `fake_db.py` to `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')` where PASSWORD in my case was 'catalog' set above.
* Setup the database by running `python /var/www/catalog/catalog/database_setup.py` and populate it with `fake_db.py`
* Disallow remove connections to the database via: `sudo nano /etc/postgresql/9.3/main/pg_hba.conf` and make sure it looks like the following:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
* Source [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)

8. And last, but not least, `sudo service apache2 restart` and launch the website!

### Bug-Squashing References and Sources!

The following links provided me with great ideas and solutions to solve the countless bugs you will most likely encounter during this project:

* [`ImportError: No module named psycopg2`: StackOverflow](https://stackoverflow.com/questions/12906351/importerror-no-module-named-psycopg2)
* [`ImportError: No module named sqlalchemy`: StackOverflow](https://stackoverflow.com/questions/10572498/importerror-no-module-named-sqlalchemy)
* [`OperationalError: (sqlite3.OperationalError) unable to open database file (Background on this error at: http://sqlalche.me/e/e3q8`: Udacity Forums!](https://discussions.udacity.com/t/operationalerror-sqlite3-operationalerror-unable-to-open-database-file-background-on-this-error-at-http-sqlalche-me-e-e3q8/605647)

* [SQLAlchemy Error Messages](http://docs.sqlalchemy.org/en/latest/errors.html#error-e3q8)