# Udacity - Linux server configuration project

## Goals:
> You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

IP address: 54.91.38.98.
## Configurations
1. Create new user -
    * ```sudo adduser grader```
    * Edit the sudoers file
    ```sudo vi /etc/sudoers.d/grader```
    * Add ```ALL=(ALL:ALL) ALL```
2. Configure key-based authentication for grader user
    * ```touch /home/grader/.ssh/authorized_keys```
    * Copy the public key generated from your local machine to authorized_keys
    * Run ```sudo vi /etc/ssh/sshd_config```
    * Change ```PermitRootLogin without-password``` line to ```PermitRootLogin no```
    * Restart ssh with ```sudo service ssh restart```
    * Now you are only able to login using ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@54.91.38.98
3. Update all currently installed packages
    * Download package lists with ```sudo apt-get update```
    * Fetch new versions of packages with ```sudo apt-get upgrade```
4. Change SSH port from 22 to 2200 (This step is not working, so I have left the port to be 22)
    - Run `sudo vi /etc/ssh/sshd_config`
    - Change the port from 22 to 2200
    - Confirm by running `ssh -i ~/.ssh/udacity_key.rsa -p 2200 root@54.91.38.98`
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
    - `sudo ufw allow 2200/tcp` (Again here, I have allowed port 22)
    - `sudo ufw allow 80/tcp`
    - `sudo ufw allow 123/udp`
    - `sudo ufw enable`
6. Configure the local timezone to UTC
    - Run `sudo dpkg-reconfigure tzdata` and then choose UTC
7. Install Apache
    - `sudo apt-get install apache2`
8. Install mod_wsgi
    - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
    - Enable mod_wsgi with `sudo a2enmod wsgi`
    - Start the web server with `sudo service apache2 start`
9. Clone the Catalog app from Github
    - Install git using: `sudo apt-get install git`
    - `cd /var/www`
    - `sudo mkdir catalog`
    - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
    - `cd /catalog`
    - Clone your project from github `git clone https://github.com/asmita0917/catalog.git`
    - Create a catalog.wsgi file, then add this inside:
    ```
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'supersecretkey'
    ```
10. Install virtual environment
    - Install the virtual environment `sudo pip install virtualenv`
    - Create a new virtual environment with `sudo virtualenv venv`
    - Activate the virutal environment `source venv/bin/activate`
    - Change permissions `sudo chmod -R 777 venv`
11. Install Flask and other dependencies
      - Install pip with `sudo apt-get install python-pip`
     - Install Flask `pip install Flask`
     - Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

12. Update path of client_secrets.json file
    - `vi __init__.py`
     - Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
13. Configure and enable a new virtual host
    1. Create a virtual host conifg file: `$ sudo nano /etc/apache2/sites-available/catalog.conf`.
    2. Paste in the following lines of code:
    ```<VirtualHost *:80>
    ServerName 54.91.38.98
    ServerAdmin asmitadutta44@gmail.com
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
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
    </VirtualHost>```
14. Install and configure PostgreSQL
    1. Install some necessary Python packages for working with PostgreSQL: `$ sudo apt-get install libpq-dev python-dev`.
    2. Install PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`.
    3. Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a tusted user who can access the database software. So let's change the user with: `$ sudo su - postgres`, then connect to the database system with `$ psql`.
    4. Create a new user called 'catalog' with his password: `# CREATE USER catalog WITH PASSWORD 'sillypassword';`.
    5. Give *catalog* user the CREATEDB capability: `# ALTER USER catalog CREATEDB;`.
    6. Create the 'catalog' database owned by *catalog* user: `# CREATE DATABASE catalog WITH OWNER catalog;`.
    7. Connect to the database: `# \c catalog`.
    8. Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`.
    9. Lock down the permissions to only let *catalog* role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`.
    10. Log out from PostgreSQL: `# \q`. Then return to the *grader* user: `$ exit`.
15. Restart Apache
    ```sudo service apache2 restart```
16. Visit site at http://54.91.38.98
