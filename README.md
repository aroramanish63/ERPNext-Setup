# Guide-to-Install-Frappe-ERPNext14-in-Ubuntu-22.04-LTS / Ubuntu-20.04
A complete Guide to Install Frappe Bench in Ubuntu 22.04 LTS and install Frappe/ERPNext Application

### Pre-requisites 
```
Python 3.10+
Node.js 16
Redis                                         (caching and real time updates)
MariaDB 10.6.x                                (to run database driven apps)
yarn 1.12+                                    (js dependency manager)
pip 22+                                       (py dependency manager)
wkhtmltopdf (version 0.12.6 with patched qt)  (for pdf generation)
cron                                          (bench's scheduled jobs: automated certificate renewal, scheduled backups)
NGINX                                         (proxying multitenant sites in production)
```
### STEP 1 Update Server
It is always a good idea to upgrade the Ubuntu package if anything is available, run the below command to upgrade and update.
```    
sudo apt update && sudo apt upgrade -y
```
Check the server's current timezone
```
date
```    
Set correct timezone as per your region
```
timedatectl set-timezone "Asia/Kolkata"
```
You can run the following command to select your timezone.
```
timedatectl list-timezones
```    
It is always recommended to reboot the server once the upgrade is done.
```
sudo reboot
```
### STEP 2  Create a new user
I am using **erpnext** as frappe-user

#### you should not use _frappe_ or _erpnext_ as a frappe-user on your production server.
```
sudo adduser erpnext
sudo usermod -aG sudo erpnext
su - erpnext
```
### STEP 3 Install git
Git is the most commonly used version control system. Git tracks the changes you make to files, so you have a record of what has been done, and you can revert to specific versions should you ever need to. Git also makes collaboration easier, allowing changes by multiple people to all be merged into one source.
```    
sudo apt install git -y
```
### STEP 4 Install python3-dev and python3-venv
Python-dev is the package that contains the header files for the Python C API, which is used by lxml because it includes Python C extensions for high performance.
```
sudo apt install python3-dev python3-venv -y
```
### STEP 5 Install setuptools and pip (Python's Package Manager).
Setuptools is a collection of enhancements to the Python distutils that allow developers to more easily build and distribute Python packages, especially ones that have dependencies on other packages. Packages built and distributed using setuptools look to the user like ordinary Python packages based on the distutils.

Pip is a package manager for Python. It's a tool that allows you to install and manage additional libraries and dependencies that are not distributed as part of the standard library.
```
sudo apt install python3-setuptools python3-pip -y
```
### STEP 6 Install Redis server
Redis can be used to process and analyze data in memory, this is prerequisite for ERPNext.
```
sudo apt install redis-server -y
```
### STEP 7 Install software-properties-common
Now install the below package to manage the repository, usually, Ubuntu 22.04 has already installed it, but for the safe side, we will run this command.
```
sudo apt install software-properties-common -y
```
If prompt for "Override local changes to /etc/pam.d/common-*?" on PAM Configuration, then safely choose "No".

### STEP 8 install wkhtmltopdf
Wkhtmltopdf is an open source simple and much effective command-line shell utility that enables user to convert any given HTML (Web Page) to PDF document or an image (jpg, png, etc).
```
sudo apt install xvfb libfontconfig1 fontconfig libxrender1 xfonts-75dpi -y
```
#### for amd64
```
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb && \
sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_amd64.deb
```    
#### for arm64
```
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_arm64.deb && \
sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_arm64.deb
```
<!--    sudo reboot

after reboot

    sudo su - erpnext
-->

### STEP 9 Install MariaDB
```
sudo apt install mariadb-server mariadb-client -y
```
<!--
IMPORTANT: During this installation you'll be prompted to set the MySQL root password.
If you are not prompted for the same You can initialize the MySQL server setup by executing the following command.

sudo service mysql start       #run this command first, only if you are running Ubuntu in a Virtual Machine.    
-->
```
sudo mysql_secure_installation
```
#### Prompt
``` 
Enter current password for root (enter for none):   (safely press Enter)
Switch to unix_socket authentication [Y/n]          (Press "Y")
Change the root password? [Y/n]                     (Press "Y")
New password:                                       ("Enter new Password")
Re-enter new password:                              ("Re-enter new Password")
Remove anonymous users? [Y/n]                       (Press "Y")
Disallow root login remotely? [Y/n]                 (Press "Y") //If Press "N" then You want to access the database from a remote server for using business analytics software like Metabase / PowerBI / Tableau, etc.
Remove test database and access to it? [Y/n]        (Press "Y")
Reload privilege tables now? [Y/n]                  (Press "Y")
```

### STEP 10  MySQL database development files
```
sudo apt install libmysqlclient-dev -y
```
### STEP 11 Edit the mariadb configuration (unicode character encoding)
You need to ensure to change the default character set of MySQL or MariaDB to Unicode instead of general. To do this you will need to edit the maria DB configuration file which is in this version located at /etc/mysql/mariadb.conf.d directory so you can directly edit this or locate the folder and then edit the file by typing the below command.
```
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Once the file opens you need to locate the line where collation-server says general and you need to modify it to Unicode as below,
```
collation-server = utf8mb4_general_ci
```
Modify above as below.
```
collation-server = utf8mb4_unicode_ci
```
Now press (Ctrl-X) and Save then exit.

And also locate my.cnf and edit the below configuration.
```
sudo nano /etc/mysql/my.cnf
```
Make sure your configuration has the below lines in the file.
```
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```
Now press (Ctrl-X) and Save then exit.

Now MySQL or MariaDB setup is now ready, let us now restart this service. You can alternatively reboot as well.
```
sudo service mysql restart
```
### STEP 12 install Node.js package
```
sudo apt install curl
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.profile
nvm install 18
```
Sometimes, due to github raw content server not responding, you have to manually download "install.sh" and run in terminal.
Download from **"https://github.com/nvm-sh/nvm/archive/refs/tags/v0.39.3.zip"**
Then unzip and copy/paste "install.sh" file in your frappe-user home directory.
Then run
```
bash install.sh
source ~/.profile
nvm install 18
```
Now it has been installed you can now check the version by typing the below command.
```
node -v
```
### STEP 13 Install Yarn using NPM
Now we will install Yarn which is a software packaging system developed by Facebook for Node.js, this is open source, so we will install it using npm.
```
npm install -g yarn
```
Now our server is ready for the installation of the frappe environment, let us now dive into the frappe environment installation.

### STEP 14 Bench Installation
The bench is a Command-line tool to manage Frappe Deployments, this tool has various commands, the frappe uses the bench for the command-line tool as well as for the bench directory, so don’t confuse yourself. We will first install the bench package which will be used to set up a frappe environment, create a site, do backup, change the setup, and so on. In short, Bench provides a user-friendly interface to set up and manage multiple frappe-based applications and sites where erpnext is one of the applications.

Now let us install the bench
```
sudo -H pip3 install frappe-bench
```
It will install a bench and will give you a message that the bench is installed successfully, now you can use various bench commands. Starting with the command "bench".

### STEP 15 Install Frappe-Bench Environment using bench CLI
Let us now create the frappe-bench environment. Here you have to decide the purpose for which you are installing ERPNext, it is just for test or training then you can use the latest version, which will be developing and may not be stable. However you can also use a stable version by choosing a specific version, You can search and learn which is the stable version today.
    
To choose a specific stable version (for Production) you can use the branch version. I will be using branch version 14 in this installation. You can look for the latest stable release of the frappe environment.
```
bench init frappe-bench --verbose --frappe-branch version-14
```
<!-- To deploy the latest (for development) frappe-bench environment make sure to run the command while you are in your home directory or your user and use the below command.

    bench init frappe-bench
-->
Now frappe bench environment is installed using bench CLI.

Now you can use various bench commands by changing the directory. So you can need to change the directory.
```
cd frappe-bench
```
Once you type "bench" you will see the various commands that bench cli has. Don’t worry, we will not be using all the commands, we just need to install EPRNext but have a quick look at these commands.

### STEP 16 ERPNext Installation on Frappe Environment

make sure that your working directory is frappe-bench.

Now you need to get the app from the frappe repository, have two options, either clone the latest development package, which is not stable currently, or install the stable package. I have given both options for you to choose from.

<!-- To install the current package which is not stable you can use the below command

    bench get-app erpnext https://github.com/frappe/erpnext
-->
Frappe and ERPNext version would be same for a proper installation. We will be installing ERPNext Version 14, for that, We will be using the below command.
```
bench get-app erpnext --branch version-14
```
Payments app is mandatory from ERPNext version-14.
```
bench get-app payments --branch version-14
```
From any of the options, it will clone the next application into the app’s directory of the frappe-bench directory. You don’t need to do anything with the directories. Just ensure that erpnext is available in the directory.

Now you have the application installed in your environment. The next step is to install the application on-site, but before that, we need to create a new site.

##### Run ```bench start``` then open another terminal window, then go to the "frappe-bench" directory and start from there

 <!--   bench start -->

To create a site, we will use the bench command as below
```
bench new-site erp.YOURDOMAIN.COM 
```
Now site is deployed, by default frappe application will be installed at site. Don’t open the site yet, because we need to install ERPNext to the site.

#### Make sure your sub-domain properly configured to access this server, where you hosted your domain.

If you have not created subdomain, then create a subdomain (preferred) erp.YOURDOMAIN.COM. Then Change DNS records as below:
```
Hostname                Type    Value               TTL
erp.YOURDOMAIN.COM      A       ERPNext-Server-IP   1800
www.erp.YOURDOMAIN.COM  A       ERPNext-Server-IP   1800
```
Run the below command to install ERPNext to the site that you have recently created
```
bench --site erp.YOURDOMAIN.COM install-app erpnext
```
it will take a few moments to install the erpnext application on your site.

Now as we created a new site, we need to make sure this is our default site, so we have to tell bench to use this site as default by using the below command
```
bench use erp.YOURDOMAIN.COM
bench start
```
Now ERPNext is installed on your server and you are ready to configure it. But before configuring there are few more steps in case you want to use this for production.

<!--
### STEP ## ERPNext Setup for Production

ERPNext only supports NGINX, so you can't use apache2 on this server. You have to remove Apache2 from your server.

sudo apt-get remove --purge apache2 apache2-data apache2-utils apache2-bin apache2.2-common

You can do the following two tests to confirm apache has been removed:

Run

which apache2

This should return a blank line.

Run

sudo service apache2 start

This should return apache2: unrecognized service
-->

### STEP 17 Production Deployment
When ERPNext is installed in development mode, you will have to restart the instance with the bench start command every time you want to access it. You can however start your instance when the server boots by setting up your instance for production.

We will be using Supervisor to manage to manage the ERPNext process and Nginx as a reverse proxy to access the ERPNext process without using port 8000

#### Process
There are two ways this can be done. I will start with the simple and straightforward one.

Run the below commands from your bench directory:
```
sudo bench setup production [BENCH USER]
sudo supervisorctl restart all
```

If you are successful, your instance should be in production mode.

If this didn’t work for you, follow the following steps:

First, change the user to your bench user and install Supervisor and Nginx with the following command:

```
su - erpnext
sudo apt-get -y install supervisor nginx
```

Next, install the frappe-bench add-on with the following command:
```
sudo pip3 install frappe-bench
```

Next, run the following command to configure ERPNext for a production environment:
```
sudo /home/erpnext/.local/bin/bench setup production erpnext
```

You should see the following output:
```
Site erpnext.example.com assigned port: 80
$ sudo /usr/bin/supervisorctl reread
erpnext-redis: available
erpnext-web: available
erpnext-workers: available
$ sudo /usr/bin/supervisorctl update
erpnext-redis: added process group
erpnext-web: added process group
erpnext-workers: added process group
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
$ sudo systemctl reload nginx
```
At this point, ERPNext is installed and configured to run on port 80. Now, open your web browser and type in Your URL without the port

## Thank You!🙏
