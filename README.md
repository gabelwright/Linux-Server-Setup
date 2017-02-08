## Setting up a Linux Server

#### IP Address: 50.112.65.170

## Setup

### - Create new user with root access:

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

### - Configure Key-Based Authentication:

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

	- `ssh grader@50.112.65.170 -i ~/.ssh/nano`

### - Configure ssh Settings:

1. Open the ssh config file

	- `sudo nano /etc/ssh/sshd_config`

2. Make the following changes:

	- change `Port 22` to `Port 2200`

	- change `PermitRootLogin yes` to `PermitRootLogin no`

	- change `PasswordAuthentication yes` to `PasswordAuthentication no`

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

3. Restart ssh so the changes will take affect:

	- `sudo service ssh restart`

4. Exit the ssh connection and re-connect using port 2200:

	- `exit`

	- `ssh grader@50.112.65.170 -i ~/.ssh/nano -p 2200`

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

### Setup Apache Server for WSGI Applications

1. Install Apache 2 and the WSGI plugin:

	- `sudo apt-get install apache2`

	- `sudo apt-get install libapache2-mod-wsgi`

	- `sudo apt-get install python-setuptools`

2. Tell apache how to respond to WSGI requests by editing the confige file:

	- `sudo nano /etc/apache2/sites-enabled/000-default.conf`

3. Add the following line right before `</VirtualHost>`

	- `WSGIScriptAlias / /var/www/html/myapp.wsgi`

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

4. Restart apache so the changes take affect:

	- `sudo apache2ctl restart`

5. Create your WSGI app file:

	- `sudo nano /var/www/html/myapp.wsgi`

6. To test if everything has been setup right, paste in the following:
		
		def application(environ, start_response):
		    status = '200 OK'
		    output = 'Hurray!!! It works!'

		    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
		    start_response(status, response_headers)

		    return [output]

7. In your web browser, navigate to the web page `50.112.65.170`. If you should see the message `Hurray!!! It works!` 

### Configure Server for flask app from Github

1. Install git and required python package:

	- `sudo apt-get install git python-dev`

2. Create the new directories to hold our app:

	- `sudo mkdir /var/www/tutor_site`

	- `sudo mkdir /var/www/tutor_site/tutor_site`

	- `sudo mkdir /var/www/tutor_site/tutor_site/static`

	- `sudo mkdir /var/www/tutor_site/tutor_site/templates`

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

8. Again, navigate to your IP address in your web browser.  If everything is set up correctly, you should see `Hello World, everything seems to be working!`

9. Create the app config file, add the following code, and then enable the app:

	- `sudo nano /etc/apache2/sites-available/tutor_site.conf`

			<VirtualHost *:80>
	            ServerName 50.112.65.170
	            ServerAdmin admin@50.112.65.170
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

	- `nano tutor_site.wsgi`

			#!/usr/bin/python
			import sys
			import logging
			logging.basicConfig(stream=sys.stderr)
			sys.path.insert(0,"/var/www/tutor_site/")

			from tutor_site import app as application
			application.secret_key = 'Add your secret key'

	- save <kbd>ctrl</kbd>+<kbd>O</kbd> and exit <kbd>ctrl</kbd>+<kbd>X</kbd>

11. Restart apache:

	- `sudo service apache2 restart`

12. Again, navigate to the IP address in your web browser and you should see `Hello World, everything seems to be working!`

### Install and setup PostgreSQL

1. Navigate to your virtual enviornment and activate it:

	- `cd /var/www/tutor_site/tutor_site`

	- `source temp_env/bin/activate`

2. Install PostgreSQL as well as other needed packages:

	- `sudo apt-get install postgresql`

	- `sudo pip install sqlalchemy` 

	- `sudo apt-get install python-psycopg2`

3. When PostgreSQL is installed, it automatically creates a new user named `postgres` you need to switch to that user and start PostgreSQL:

	- `su - postgres`

	- `psql`

4. Now that we are in Postgres, we can manipulate our database.  First create a new user with permission to create databases:

	- `CREATE USER catalog WITH PASSWORD 'tutor';`

	- `ALTER USER catalog CREATEDB;`

	- `CREATE DATABASE tutor WITH OWNER catalog;`

5. Switch to our new user and revoke public access to the new database:

	- `REVOKE ALL ON SCHEMA public FROM public;`

	- `GRANT ALL ON SCHEMA public TO catalog;`

### Add Application from Github

1. Clone your application from git:

	- `cd /var/www/tutor_site`

	- `git clone https://github.com/acronymcreations/tutor_site.git`

	- `mv /var/www/tutor_site /var/www/tutor_site/tutor_site`

2. Rename the main python file:

	- `mv /var/www/tutor_site/tutor_site/main.py /var/www/tutor_site/tutor_site/__init__.py`

3. (Optional) If your flask app uses SQLite, it should be switched to PostgreSQL:

	a. Open the database setup file and make the following changes:

	- `cd /var/www/tutor_site/tutor_site`

	- `nano db_setup.py`

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

4. Edit your filepaths to match their new location:

	- `cd /var/www/tutor_site/tutor_site`

	- `nano __init__.py`

	- change

		CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']

		to 

		CLIENT_ID = json.loads(open('/var/www/tutor_site/tutor_site/client_secrets.json', 'r').read())['web']['client_id']

		and 

		oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')

		to

		oauth_flow = flow_from_clientsecrets('/var/www/tutor_site/tutor_site/client_secrets.json', scope='')




































