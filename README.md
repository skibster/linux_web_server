# Linux Web Server

This is a summary of how I configured the Amazon EC2 instance to run the item_catalog application.

## SSH access

 - SSH is available on port 2200 at the IP address: 52.24.117.220.

## Website URL access

 - The public URL for the hosted website is: http://ec2-52-24-117-220.us-west-2.compute.amazonaws.com or just 52.24.117.220

## Software installation summary and configuration

### Make root RSA private key

In the terminal on the local machine, create the RSA private key file by running the following commands:

`touch ~/.ssh/udacity_key.rsa`
`nano ~/.ssh/udacity_key.rsa`

then, pasted the contents of the udacity_key.rsa file you downloaded from the Udacity website.

### Make ssh public/private key for the user 'grader'

In the terminal on the local machine, create the RSA public/private key file by running the following commands:

`ssh-keygen`

When asked to save the key, enter: `~/.ssh/grader.rsa`
`cat ~/.ssh/grader.rsa.pub` and copy the contents of this output to the clipboard.

### ssh to the remote server

`ssh -i ~/.ssh/udacity_key.rsa root@52.24.117.220`

### update and upgrade the server

`sudo apt-get update`
`sudo apt-get upgrade`

### Install finger to get user information when adding new users

`sudo apt-get install finger`

### Add the user grader

`adduser grader`

### Setup the RSA public key for grader

`mkdir /home/grader/.ssh`
`touch /home/grader/.ssh/authorized_keys`
`nano /home/grader/.ssh/authorized_keys`

Paste the contents of the clipboard from above and save the file.

### Add user grader to sudoers

`touch /etc/sudoers.d/grader`
`nano /etc/sudoers.d/grader`

### Add the following to grader file to make them a sudoer

`grader	ALL=(ALL:ALL) ALL`

### Install NTP

`sudo apt-get install ntp`

### Set server time to UTC

`sudo dpkg-reconfigure tzdata`

Choose None of the above, on next screen, choose UTC.

### Change SSH port to 2200 

`sudo nano /etc/ssh/sshd_config` 

Change `Port 22` to `Port 2200` # this makes ssh run on port 2200 instead of 22.
Add `AllowUsers grader` # this makes grader the only user that can ssh to the remote server
`sudo service ssh restart` # restart the ssh service.

If all goes well, you will get disconnected, but should be able to log back in using the following command:

`ssh -p 2200 -i ~/.ssh/grader.rsa grader@52.24.117.220`

### Setup and activate the UFW (Uncomplicated Firewall)
`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
`sudo ufw allow 2200/tcp`
`sudo ufw allow www`
`sudo ufw allow ntp`

WARNING: This next command enables the firewall. Before running it, make sure you have changed the ssh port to 2200 AND that you've opened port 2200 in the firewall. Otherwise, you will be permanently locked out of accessing your server.

`sudo ufw enable`

### Install Apache HTTP server

`sudo apt-get install apache2`
`sudo apt-get install python-setuptools libapache2-mod-wsgi python-dev`

### Install git

Install git so that you can clone the item_catalog application into the Flask app.

`sudo apt-get install git`

### Install the Flask application

Follow these [instructions](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) to deploy the Flask application, using `git clone` to clone the item_catalog app from GitHub to the remote server.

In the .git folder, insert a .htaccess file to deny access to that directory, though I don't know if this is necessary or not.

 `sudo nano .htaccess`

Type: `deny from all` and save the file.

Once everything is configured, you'll need to restart the apache2 service: `sudo apache2ctl restart`

### Install postgres

Follow these [instructions](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04) to install and configure the postgres database server and create the user: catalog and create the database called catalog.

### Install automatic security updates

Follow these [instructions](https://help.ubuntu.com/community/AutomaticSecurityUpdates) for using unattended-upgrades to keep server security software automatically updated.

### Install Fail2Ban 

Fail2Ban can be setup to block IP addresses for users attempting to gain access to the server via ssh who fail authentication 5 times.

Follow these [instructions](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04) to install Fail2Ban.  Configure Fail2Ban to send email to grader@localhost. To read the emails in the grader account, you'll need to install mailutils.

`sudo apt-get install mailutils`


### Install Glances

To monitor the health of the server and its processes, install the  glances application.

`sudo apt-get install glances`

To run glances, simply enter the command `glances`.
