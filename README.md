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

	- `touch authorized_keys`

4. Open this file and copy/paste the public key from your local machine (step 2) into it

	- `nano authorized_keys`

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



































