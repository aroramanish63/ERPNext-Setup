#### ERPNext 14 Setup issues coming while doing installation

After successful installation erpnext 14 on server but having some issues which is listed here:

#### 1. When bench has not been started using bench start command in development mode ?
Solution: downgrade the bench version to version 5.10 by using the command
```
sudo pip3 install frappe-bench==5.12.0
```

#### 2. Production setup commands, follow below link.
https://frappeframework.com/docs/v14/user/en/bench/guides/setup-production#supervisor

#### 3. Unable to start my bench - bind: Address already in use.
Solution:
find ports using below command
```
 $ lsof -i -P -n | grep LISTEN
```
kill the listed ports (11000, 12000, 13000, 8000, 9000) using below command
```
sudo lsof -t -i tcp:<PORT> -s tcp:listen | sudo xargs kill
```

#### 4. Error: <class ‘PermissionError’>, [Errno 13] Permission denied: file: /usr/lib/python3/dist-packages/supervisor/xmlrpc.py line
```
$ sudo nano /etc/supervisor/supervisord.conf
```
(Add these lines under [unix_http_server])
```
chmod=0760
chown=frappe:frappe
```
you might also need to restart supervisor
```
sudo -A systemctl restart supervisor
```

if its Still didn’t work?

Create a group called supervisor, and add user into that group
```
sudo groupadd supervisor
sudo usermod -a <user> -G supervisor
```

Now edit supervisor.conf
```
sudo nano /etc/supervisor/supervisor.conf
```

and add as follows:

```
[unix_http_server]
file=/var/run/supervisor.sock ; (the path to the socket file)
chmod=0770 ; socket file mode (default 0700)
chown=<user>:supervisor
```

then restart supervisord
```
sudo supervisorctl reload
```
or
```
sudo service supervisor restart
```

#### 5. Getting 404 not found on bench start
Enter in your frappe-bench project directory, after that follow below steps

```
$ cd sites
$ sudo touch currentsite.txt
$ sudo nano currentsite.txt
```
put your erp sitename in this file and save it. after that in your frappe bench project directory
```
$ bench start
```

ERPNext 14 setup will work.

#### 6. Reset admin password
```
$ bench --site [site] set-admin-password [new password]
```

#### 7. Configuring the Firewall for Multitenant site or port enable settings
Although configuring a firewall for development is optional, for production it is a mandatory security practice.

You will need to open the following ports on your ERPNext server:

80/tcp and 443/tcp for HTTP and HTTPS respectively
3306/tcp for MariaDB connection (recommended only if you need remote access to database)
143/tcp and 25/tcp for IMAP and STMP respectively
22/tcp for SSH (if you have not already enabled OpenSSH in your UFW settings)
8000/tcp for testing your platform before deploying to production

To open multiple ports at once you can use the following command:
```
sudo ufw allow 22,25,143,80,443,3306,8000/tcp
```

Alternatively, you can allow connections from specific IP addresses on specific ports using this command:
```
sudo ufw allow from server_IP to any port port_number
```

After opening all necessary ports enable the firewall:
```
sudo ufw enable
```

Now confirm the status of your firewall:
```
sudo ufw status
```

