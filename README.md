# Gitea Raspberry Howto

This guide explains howto install Gitea on a Raspberry Pi. Further information about Gitea can be found at:

https://gitea.io

https://github.com/go-gitea

## Pre-requisites
It assumes that you have a RaspBerry Pi with RaspBian installed. The howto guide is validated RaspBerry Pi B+ 1.2 but should on other versions as well.

Let's dig into the guide.

## Howto

### Prepare your OS
Make sure your Raspbian is up to date by running the following commands on your RaspBarry Pi:

````
sudo apt-get update
sudo apt-get upgrade
````
### Install a database
Gitea reqiures a database to handle user and system data. You can choose between MariaDB, PostgreSQL, SQlite3, MSSQL. For this installation we will use PostgreSQL.

> Note! Version 10.1 of MariaDB is not compatible with Gitea version 1.5.1. Please follow the issue https://github.com/go-gitea/gitea/issues/2979 if you plan to install MariaDB.

Install PostgreSQL by using the following command:

````
sudo apt-get install postgreSQL
````
Open PostgreSQL terminal to create a user for Gitea.
````
sudo -u postgres psql -d template1
````
````
CREATE USER gitea CREATEDB;
````
To set the password for user gitea, type:
````
\password gitea
````
You will be asked to enter a password and password confirmation. Keep the password because you will need it for the configuration of Gitea.

Create a new database for Gitea:
````
CREATE DATABASE gogs OWNER gogs;
````
Exit the PostgreSQL terminal:
````
\q
````
### Install Gitea
First we start by creating a new user under which we will run the Gitea service. Use –disabled-login as we don't want to use it for login into the Raspberry Pi. Use –gecos to allow us to set a name for the user, “gitea“.
````
sudo adduser --disabled-login --gecos 'Gitea' git
````
Switch to the newly created user:
````
sudo su git
````
Change to the home directory of user git and create a new directory where we will store the Gitea binaries. We also switch to the new directory.
````
cd ~
mkdir gitea
cd gitea
````
Now we need to download the correct Gitea binaries. First go to https://dl.gitea.io/gitea/ and pick the latest version for your specific RaspBerry Pi. Copy the link of the actual file, in this case https://dl.gitea.io/gitea/1.5.1/gitea-1.5.1-linux-arm-6 and replace it into the following command.
````
wget https://dl.gitea.io/gitea/1.5.1/gitea-1.5.1-linux-arm-6 -O gitea
````
### Run Gitea as a service

To be able to run it as a service, we first need to give execution rights to the file for the user git. Finally we exit from user `git` and return to the user `pi`.
````
chmod +x gitea
exit
````
This service will have Gitea lunched automatically at startup and we can also easily stop and start Gitea. You can either copy this file: gitea.service /etc/systemd/system/ or create a service file with the following command:
````
sudo nano /etc/systemd/system/gitea.service
````
Copy the collowing content to the file:
````
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target

[Service]
# Modify these two values ​​and uncomment them if you have
# repos with lots of files and get to HTTP error 500 because of that
###
# LimitMEMLOCK=infinity
# LimitNOFILE=65535
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/home/git/gitea
ExecStart=/home/git/gitea/gitea web
Restart=always
Environment=USER=git 
HOME=/home/git

[Install]
WantedBy=multi-user.target
````
Save the file with `Ctrl+x` followed by `y` and then `enter`.

If everything is correct we should now be able to enable and finally start our service:
````
sudo systemctl enable gitea.service
sudo systemctl start gitea.service
````
### Configuring Gitea
Now we will continue to configure Gitea. This is done via the Gitea web application. Start your web browser and enter you IP adress of your RaspBerry Pi followed by the port 3000, for example:
````
http://192.168.0.143:3000
````
 - [ ] TODO: Complete the first time installation guide.

Optional TODO's
 - [ ] Install and configure nginx as reverse proxy
 - [ ] Creat backup script
 - [ ] Create update process for Gitea

