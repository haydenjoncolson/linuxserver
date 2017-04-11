# Linux Server Configuration on Amazon Lightsail

## Live Version
34.203.129.144

## Configuration

1. ssh into your server as ubuntu user
    * Download the private key from the account page
    * Copy the file to your local /.ssh/ directory
        ```bash
        cp ~/Downloads/LightsailDefaultPrivateKey.pem ~/.ssh/Udacity.pem
        ```
    * change permissions for user ubuntu
        ```bash
        chmod 600 ~/.ssh/Udacity.pem
        ```
    * ssh into server using private key
        ```bash
        ssh ubuntu@YourPublicIP -i ~/.ssh/Udacity.pem
        ```
2. Add new user grader with password grader
    ```bash
    sudo adduser grader
    ```
    * install finger to check if user grader is there
        ```bash
        sudo apt-get install finger
        finger grader
        ```
3. Give grader sudo access
    * Copy sudoers.d file for grader
        ```bash
        cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
        ```
    * Change ubuntu to grader
        ```bash
        sudo nano /etc/sudoers.d/grader
        ```
    * Copy root line, paste below and change root to grader
        ```bash
        sudo visudo
        ```
4. Generate keygen in your terminal's home directory
    ```bash
    sudo ssh-keygen
    ```
    * Enter /.ssh/LinuxServerProject for directory

5. Login as grader
    ```bash
    sudo login grader
    ```
    * run these commands to create an .ssh directory
        ```bash
        mkdir .ssh
        touch .ssh/authorized_keys
        ```
    * Copy the contents of the public key generated by the keygen
        ```bash
        cat ~/.ssh/LinuxServerProject.pub
        ```
    * Paste into authorized keys
        ```bash
        sudo nano .ssh/authorized_keys
        ```
    * Change permissions by running
        ```bash
        chmod 700 .ssh
        chmod 600 .ssh/authorized_keys
        ```
    * Exit ssh and login as grader with public key
        ```bash
        ssh grader@YourPublicIP -i ~/.ssh/LinuxProjectServer
        ```
    *  Disable root login
        ```bash
        sudo nano /etc/ssh/sshd_config
        ```
        Change to PermitRootLogin no

6. Change Port Configuration
    * Login as ubuntu and grader into separate terminals
    * As grader, change port configuration from Port 22 to Port 2200
        ```bash
        sudo nano /etc/ssh/ssh_config
        ```
    * restart ssh
        ```bash
        sudo service ssh restart
        ```
    * On the Lightsail Networking Page, add a custom tcp protocol for port 2200

7. Firewall Configuration
  * By default, block all incoming connections on all ports:
  ```bash
  sudo ufw default deny incoming
  ```

  * Allow outgoing connection on all ports:
  ```bash
  sudo ufw default allow outgoing
  ```

  * Allow incoming connection for SSH on port 2200:
  ```bash
  sudo ufw allow 2200/tcp
  ```

  * Allow incoming connections for HTTP on port 80:
  ```bash
  sudo ufw allow www
  ```

  * Allow incoming connection for NTP on port 123:
  ```bash
  sudo ufw allow ntp
  ```

  * To enable the firewall, use:
  ```bash
  sudo ufw enable
  ```

  * To check the status of the firewall, use:
  ```bash
  sudo ufw status
  ```

[Source 1-6](https://docs.google.com/document/d/1pvE6os2ctLevO_EBmg3Leq4VEc1DbORcxQ_zpPqVr78/edit?usp=sharing)

8. Update all currently installed packages
    ```bash
    sudo apt-get update
    sudo apt-get upgrade
    ```
9. Install Apache, mod_wsgi, and postgreql
    ```bash
    sudo apt-get install apache2
    sudo apt-get install libapache2-mod-wsgi python-dev
    sudo apt-get install postgresql postgresql-contrib
    ```
    * Disallow Remote Connections
        ```bash
        sudo nano /etc/postgresql/9.5/main/pg_hba.conf
        ```
        Make sure it looks like this:
        ```bash
        local   all             postgres                                peer
        local   all             all                                     peer
        host    all             all             127.0.0.1/32            md5
        host    all             all             ::1/128                 md5
        ```
      [Source](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
    * Enable mod_wsgi and create catalog flask app
    ```bash
    sudo a2enmod wsgi
    sudo mkdir catalog
    cd catalog
    sudo mkdir catalog
    cd catalog
    sudo apt-get install python-pip
    sudo pip install virtualenv
    sudo virtualenv venv
    source venv/bin/activate
    ```

10. Configure Postgreql Database
    * Create user for catalog
        ```SQL
        CREATE USER catalog;
        ```
    * Create database catalog with catalog user as owner
        ```SQL
        CREATE DATABASE catalog WITH OWNER catalog;
        ```
    * Connect to database
        ```SQL
        \c catalog;
        ```
    * Revoke schema from public and grant to catalog user.
        ```SQL
        REVOKE ALL ON schema public from public;
        GRANT ALL ON  schema public to catalog;
        ```

11. Install Git
    ```bash
    sudo apt-get install git
    ```
12. Clone item catalog into /var/www/catalog
    ```
    git clone https://github.com/haydenjoncolson/itemcatalog
    ```
13. Copy item catalog into /var/www/catalog/catalog
    ```bash
    cp /var/www/catalog/itemcatalog/var/www/catalog/catalog
    ```
14. Make the .git directory not publicly accessible
    ```bash
    cd /var/www/catalog/  
    sudo nano .htaccess and add RedirectMatch 404 /\.git
    ```    
15. Install Item Catalog Modules
    ```bash
    sudo apt-get install python-psycopg2 python-flask
    sudo apt-get install python-sqlalchemy python-pip
    sudo pip install oauth2client
    sudo pip install requests
    sudo pip install httplib2
    ```
16. Change the engines in the database_setup, items, and itemcatalog python
    ```bash
    engine = create_engine('postgresql://catalog:catalog@34.203.129.144/catalog')
    ```

17. Configure virtual host
    ```bash
    sudo nano /etc/apache2/sites-available/catalog.conf
    ```
    ```bash
    <VirtualHost *:80>
		ServerName Public-IP
		ServerAdmin admin@Public-IP
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
    * enable site
    ```bash
    sudo a2ensite catalog
    ```
18. Create the wsgi file
    ```python
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog")

    from catalog import app as application
    from catalog.database_setup import setup_db
    from caatalog.items import populate
    application.secret_key = 'super_secret_key'  

    application.config['DATABASE_URL'] = 'postgresql://catalog:catalog@34.203.129.1$

    # Create database and populate it, if not already done so.
    setup_db(application.config['DATABASE_URL'])
    populate()
    ```

19. Run sudo python itemcatalog.py
