## Setting up a Linux Server

#### IP Address: 52.38.86.34

## Setup

### Create new user with root access:

1. ssh into server using the provided rsa key

2. Create a new user

	- `adduser grader`

		- Assign genertic password (passwords will be disabled so it will not be used)

3. Give `grader` root access by creating a sudoers config file:

	- `nano /etc/sudoers.d/grader`

4. Add the following line

	- `grader ALL=(ALL) NOPASSWD:ALL`

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

5. Switch to the new user

	- `su - grader`

6. Upgrade installed packages:

	- `sudo apt-get update`

	- `sudo apt-get upgrade`

### Configure Key-Based Authentication:

1. On your local machine, run the folowing command to generate a new rsa key for `grader.` For name and location, use `~/.ssh/nano`

	- `ssh-keygen`

2. Print out the contents of the public key

	- `cat ~/.ssh/nano`

3. In the ssh terminal, create a new directory for the key

	- `mkdir .ssh`

	- `touch .ssh/authorized_keys`

4. Open this file and copy/paste the public key from your local machine (step 2) into it

	- `nano .ssh/authorized_keys`

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

5. Configure the file permissions to allow access

	- `cd`

	- `sudo chmod 700 .ssh`

	- `sudo chmod 644 .ssh/authorized_keys`

6. Close the connection and then reconnect using the rsa key

	- `exit`

	- `ssh grader@52.38.86.34 -i ~/.ssh/nano`

### Configure ssh Settings:

1. Open the ssh config file

	- `sudo nano /etc/ssh/sshd_config`

2. Make the following changes:

	- change `Port 22` to `Port 2200`

	- change `PermitRootLogin yes` to `PermitRootLogin no`

	- change `PasswordAuthentication yes` to `PasswordAuthentication no`

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

3. Restart ssh so the changes will take affect:

	- `sudo service ssh restart`

4. (Optional) If `sudo: unable to resolve host ip-XX-XX-XX-XXX` appears after using the `sudo` command:

	- `cd /etc`

	- `sudo nano hosts`

	- change the first line so it reads `127.0.0.1 localhost ip-XX-XX-XX-XXX`

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

5. Exit the ssh connection and re-connect using port 2200:

	- `exit`

	- `ssh grader@52.38.86.34 -i ~/.ssh/nano -p 2200`

### Configure the Firewall

1. Disable all incoming traffic and allow all outgoing traffic:

	- `sudo ufw default allow outgoing`

	- `sudo ufw default deny incoming`

2. Allow connections for ssh, HTTP, and NTP:

	- `sudo ufw allow 2200`

	- `sudo ufw allow 123`

	- `sudo ufw allow 80`

3. Turn on firewall:

	- `sudo ufw enable`

### Configure Timezone to UTC

1. Execute the following command:

	- `sudo dpkg-reconfigure tzdata`

2. Scroll to the bottom and select `None of the above`

3. Scroll down and select `UTC`

### Configure Server for flask

1. Install Apache 2 and the WSGI plugin:

	- `sudo apt-get install apache2`

	- `sudo apt-get install libapache2-mod-wsgi python-dev`

	- `sudo apt-get install python-setuptools`

	- `sudo a2enmod wsgi` 

2. Create the new directories to hold our app:

	- `sudo mkdir /var/www/tutor_site`

	- `sudo mkdir /var/www/tutor_site/tutor_site`

3. Move into the directory you just created and create a new file to hold all of our python logic:

	- `cd /var/www/tutor_site/tutor_site`

	- `sudo nano __init__.py`

4. For now, paste in a simple 'Hello World' program to ensure everything works:

		from flask import Flask
		app = Flask(__name__)
		@app.route("/")
		def hello():
		    return "Hello World, everything seems to be working!"
		if __name__ == "__main__":
			app.run()

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

5. Install pip and virtualenv:

	- `sudo apt-get install python-pip`

	- `sudo pip install virtualenv`

6. Create a new virtual environment and start it:

	- `sudo virtualenv temp_env`

	- `source temp_env/bin/activate`

7. Install Flask and start flask app:

	- `sudo pip install Flask`

	- `sudo python __init__.py`

8. If everything is working correctly, you should see `Running on http://localhost:5000/` in the terminal. Next deactivate your envirnment:

	- <kbd>ctrl</kbd>+<kbd>C</kbd>

	- `deactivate`

9. Create the app config file, add the following code, and then enable the app:

	- `sudo nano /etc/apache2/sites-available/tutor_site.conf`

			<VirtualHost *:80>
	            ServerName 52.38.86.34
	            ServerAdmin admin@52.38.86.34
	            WSGIScriptAlias / /var/www/tutor_site/tutor_site.wsgi
	            <Directory /var/www/tutor_site/tutor_site/>
	                    Order allow,deny
	                    Allow from all
	            </Directory>
	            Alias /static /var/www/tutor_site/tutor_site/static
	            <Directory /var/www/tutor_site/tutor_site/static/>
	                    Order allow,deny
	                    Allow from all
	            </Directory>
	            ErrorLog ${APACHE_LOG_DIR}/error.log
	            LogLevel warn
	            CustomLog ${APACHE_LOG_DIR}/access.log combined
			</VirtualHost>

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

	- `sudo a2ensite tutor_site`

	- `service apache2 reload`

10. Create the wsgi file and add the following code:

	- `cd /var/www/tutor_site`

	- `sudo nano tutor_site.wsgi`

			#!/usr/bin/python
			import sys
			import logging
			logging.basicConfig(stream=sys.stderr)
			sys.path.insert(0,"/var/www/tutor_site/")

			from tutor_site import app as application
			application.secret_key = 'something'

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

11. Restart apache:

	- `sudo service apache2 restart`

12. Navigate to the IP address in your web browser and you should see `Hello World, everything seems to be working!`

### Install and setup PostgreSQL

1. Navigate to your virtual enviornment and activate it:

	- `cd /var/www/tutor_site/tutor_site`

	- `source temp_env/bin/activate`

2. Install PostgreSQL as well as other needed packages:

	- `sudo apt-get install postgresql`

	- `sudo pip install sqlalchemy` 

	- `sudo apt-get install python-psycopg2`

3. When PostgreSQL is installed, it automatically creates a new user named `postgres` you need to switch to that user and start PostgreSQL:

	- `sudo -u postgres psql`

4. Now that we are in Postgres, we can manipulate our database.  First create a new user with permission to create databases:

	- `CREATE USER catalog WITH PASSWORD 'tutor';`

	- `ALTER USER catalog CREATEDB;`

	- `CREATE DATABASE tutor WITH OWNER catalog;`

5. Switch to our new user and revoke public access to the new database:

	- `REVOKE ALL ON SCHEMA public FROM public;`

	- `GRANT ALL ON SCHEMA public TO catalog;`

### Add Application from Github

1. Clone your application from git:

	- `sudo apt-get install git`

	- `cd /var/www/tutor_site`

	- `git clone https://github.com/acronymcreations/tutor_site.git`

	- `mv /var/www/tutor_site /var/www/tutor_site/tutor_site`

2. Rename the main python file:

	- `mv /var/www/tutor_site/tutor_site/main.py /var/www/tutor_site/tutor_site/__init__.py`

3. (Optional) If your flask app uses SQLite, it should be switched to PostgreSQL:

	a. Open the database setup file and make the following changes:

	- `cd /var/www/tutor_site/tutor_site`

	- `sudo nano db_setup.py`

	- change 

			engine = create_engine('sqlite:///localtutors.db')

			to

			engine = create_engine("postgresql://catalog:tutor@localhost/tutor")

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

	b. Make a similar change in your `__init__.py` file:

	- `nano __init__.py`

	- change

			engine = create_engine('sqlite:///localtutors.db')

			to

			engine = create_engine("postgresql://catalog:tutor@localhost/tutor")

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

3. Enter your virtual environment and install all required packages:

	- `service temp_env/bin/activate`

	- `sudo pip install httplib2`

	- `sudo pip install requests`

	- `sudo pip install oauth2client`

4. Add your client secrets and edit their filepaths to match their new location:

	- `cd /var/www/tutor_site/tutor_site`

	- `sudo nano client_secrets.json`

		Copy/paste your client secrets from google.

	- `sudo nano fb_client_secrets.json`

		Copy/paste your client secrets from facebook.

	- `nano __init__.py`

	- change

			CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']

			to 

			CLIENT_ID = json.loads(open('/var/www/tutor_site/tutor_site/client_secrets.json', 'r').read())['web']['client_id']

	- change

			oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')

			to

			oauth_flow = flow_from_clientsecrets('/var/www/tutor_site/tutor_site/client_secrets.json', scope='')

	- change

			app_id = json.loads(open('fb_client_secrets.json', 'r').read())['web']['app_id']

			to 

			app_id = json.loads(open('/var/www/tutor_site/tutor_site/fb_client_secrets.json', 'r').read())['web']['app_id']

	- change

			app_secret = json.loads(open('fb_client_secrets.json', 'r').read())['web']['app_secret']

			to

			app_secret = json.loads(open('/var/www/tutor_site/tutor_site/fb_client_secrets.json', 'r').read())['web']['app_secret']

	- change

			UPLOAD_FOLDER = '/vagrant/tutor_site/static/pictures/'

			to

			UPLOAD_FOLDER = '/var/www/tutor_site/tutor_site/static/pictures/'

5. Log into the google and facebook developers console and add your new web address to the accepted list of orgins and redirects.

	- If you don't have a web address for your ip, you can obtain one at [http://whatismyipaddress.com/ip-hostname](http://whatismyipaddress.com/ip-hostname)

6. Add your web domain to your `tutor_site.conf` file:

	- `sudo nano /etc/apache2/sites-available/tutor_site.conf`

	- add `ServerAlias YOUR_DOMAIN_NAME` directly under the line `ServerAdmin admin@52.38.86.34`

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

6. Prevent file indexing from a web browser:

	- `sudo a2dismod autoindex`

7. Restart apache2 one last time and enjoy your new website!

	- `sudo service restart apache2`

	- In a webbrowser, navigate to your new website!

8. If you run into errors or get an `Internal Error` message, check the logs to see what the problem is:

	- `sudo cat /var/log/apache2/error.log`































