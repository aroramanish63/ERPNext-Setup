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

