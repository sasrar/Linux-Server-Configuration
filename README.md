# Linux-Server-Configuration
A baseline installation of a Linux distribution on a virtual machine that hosts a web applications securely

# Virtual Machine Connection
<b>IP Address:</b> 52.34.104.198 <br>
<b>SSH port:</b> 2200 <br>
<b>URL:</b> <a href="http://http://ec2-52-34-104-198.us-west-2.compute.amazonaws.com">http://http://ec2-52-34-104-198.us-west-2.compute.amazonaws.com</a><br>

# Software Installation and Configuration
Before logging in I used ssh-keygen to generate a public/private key pair. I placed the private key under ~/.ssh.
Once logged into my development machine as root user these are the next steps I took:
* Installed git using: <code> sudo apt-get install git </code>
* Added new user: <code>adduser --gecos "" grader</code>
* Placed public key under authorized keys in the grader's directory: ~/.ssh/authorized_keys
* Changed sudoers file
  * Replaced <code>root ALL=(ALL:ALL) ALL</code> with <code>grader ALL=(ALL:ALL) NOPASSWD: ALL</code>
  * This disables root user login and enables grader user login
* Edited /etc/ssh/sshd_config
  * Added <code>AllowUsers grader</code> to disallow anyone but grader from using ssh to login
  * Disabled password login: <code>PasswordAuthentication no</code>
  * Changed port from 22 to 2200: <code>Port 2200</code>
* Updated application to latest version
  * <code> sudo apt-get update </code>
  * <code> sudo apt-get upgrade </code>
* Configured firewall
  * <code> ufw allow 123/udp </code>
  * <code> ufw allow 2200/tcp </code>
  * <code> ufw allow 80/tcp </code>
  * <code> ufw enable </code>
* Installed ntp
  * <code>sudo apt-get install -y ntp</code>
* Installed Flask, Sqlalchemy, Psychopg2, Apache2, other prereqs
  * <code> apt-get -y install python-flask python-sqlalchemy </code>
  * <code> apt-get -y install python-psycopg2 </code>
  * <code> apt-get -y install python-pip </code>
  * <code> apt-get -y install python2.7-dev </code>
  * <code> apt-get -y install apache2 </code>
  * <code> apt-get -y install libapache2-mod-wsgi </code>
  * <code> pip install oauth2client </code>
* Configure application directory and add wsgi file
  * made directory ItemCatalog in /var/www/
  * used git to clone ItemCatalog project into /var/www/ItemCatalog
  * Added itemCatalog.wsgi to /var/www/ItemCatalog/
  * Added the following to itemCatalog.wsgi
  <br>
  ```python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/ItemCatalog/ItemCatalog/")
  from application import app as application
  application.secret_key = 'super_secret_key'
  ```
* Added ItemCatalog.conf to /etc/apache2/sites-available/
  * Added the following to the file:
  ```xml
  <VirtualHost *:80>
     ServerAdmin sasrar6@gmail.com
     WSGIScriptAlias / /var/www/ItemCatalog/itemCatalog.wsgi
     <Directory /var/www/ItemCatalog/ItemCatalog/>
        Order allow,deny
        Allow from all
     </Directory>
     Alias /static /var/www/ItemCatalog/ItemCatalog/static
     <Directory /var/www/ItemCatalog/ItemCatalog/static/>
        Order allow,deny
        Allow from all
     </Directory>
     ErrorLog ${APACHE_LOG_DIR}/error.log
     LogLevel info
     CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
  ```
* Enable ItemCatalog site and disable default site
  * <code> a2ensite ItemCatalog </code>
  * <code> a2dissite 000-default </code>

* Installed Postgresql
  * <code> apt-get install -y python-software-properties software-properties-common postgresql-9.3 postgresql-client-9.3 postgresql-contrib-9.3 </code>

* Setup database
  * Changed user to postgres: <code>sudo su - postgres</code>
  * Started db server: <code> sudo /etc/init.d/postgresql start </code>
  * Created new user catalog with limited permissions: <code> psql --command "CREATE USER catalog WITH CREATEDB LOGIN PASSWORD 'password';" </code>
  * Created catalog db with owner catalog: <code> createdb -O catalog catalog </code> 
  * No other users can modify public schema of catalog: <code> psql -U postgres -d catalog -c "REVOKE ALL ON SCHEMA public FROM public;" </code>
  * Only catalog user can modify catalog db: <code> psql -U postgres -d catalog -c "GRANT ALL ON SCHEMA public TO catalog;" </code>

* Coding changes in ItemCatalog project
  * Changed db connection string command from sqlite to postgresql:
    * <code>  create_engine('sqlite:///catalogItems.db') </code> to <code> create_engine('postgresql://catalog:password@localhost/catalog') </code>
  * Had to specify full path to client_secrets.json file: /var/www/ItemCatalog/ItemCatalog/client_secrets.json
* Adding credentials to Google developer console:
  * Added new url under ItemCatalog>Credentials>Authorized Javascript origins

# List of third part resources
1. Git
2. Flask
3. Sqlalchemy
4. Psychopg2
5. Apache2
6. Oauth2Client
