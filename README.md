IP and SSH port
----------------
52.35.199.115, 2200

rsa key setup
----------------
Once udacity_key.rsa is downloaded and this is copied into local user .ssh directory,
It is necessary to change key permissions. This is done with:
```sh
$ chmod 600 ~/.ssh/udacity_key.rsa
```
Now it is possible to connect to the server as root  with this command
```sh
$ ssh -i ~/.ssh/udacity_key.rsa root@xxx.xxx.xxx.xxx
```

Creating grader user
--------------------
Once into server as root, user grader in created:
```sh
root$ adduser grader
```
System will ask about user password and other related information.
Final confirmation is required aswell. Because password authentication is not
enabled into `/etc/ssh/sshd_config`, It wont be possible to connect to server as
grader just with user and password entry. It is required to add a public key for
grader.

Creating a public key for ssh connection as grader
---------------------------------------------------
From our local machine shell, type:
```sh
$ ssh-keygen
```
Key has been saved into my home .ssh home directory as fullstack. 
```sh
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/sergio/.ssh/id_rsa): /home/localusr/.ssh/fullstack
```
Once again connect to the server as root. It is possible to change to grader user with:
```sh
root$ su - grader
```
Now previous generated key content must be copied into grader .ssh directory, in a file called  authorized_keys. If these do not exist, these must be created into
grader home directory
```sh
grader$ mkdir .ssh
grader$touch .ssh/authorized_keys
```

Back to our local machine we copy the previous generated key content

```sh
$ cat .ssh/fullstack
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
...xxxxxxxxxxxx
```
Go Back to remote server grader session and paste into `grader .ssh/authorized_keys` file. Open `grader .ssh/authorized_keys` with
```sh
grader$ nano .ssh/authorized_keys
```


Save this change. Another key will have to be generated and copied (added) for any local machine which a remote grader session is intended to be initiated from. It is important to update .ssh and .ssh/authorized_keys permissions. This is done with:
```sh
grader$ chmod 700 .ssh
grader$ chmod 644 .ssh/authorized_keys
```

Now it is possible to connect from the local machine to the remote server as grader user:
```sh
$ ssh grader@xxx.xxx.xxx.xxx -i ~/.ssh/fullstack
```

Giving Sudo Access to grader user
---------------------------------
Connect to server as root:
```sh
$ ssh -i ~/.ssh/udacity_key.rsa root@xxx.xxx.xxx.xxx
```
Create this file `/etc/sudoers.d/grader`
```sh
root$ nano /etc/sudoers.d/grader
```
And add this content:
`grader ALL=(ALL) NOPASSWD:ALL`
Save this change

Now it is possible to `sudo` from grader session


Updating all currently installed packages
-----------------------------------------
```sh
$ sudo apt-get update
$ sudo apt-get upgrade
```

Changing the SSH port from 22 to 2200
--------------------------------------

For changing ssh service listenign port, it is required to edit `/etc/ssh/sshd_config`.
Into this file change `Port 22` to `Port 2200`. 
Restart SSH service:
```sh
$ sudo service ssh restart
```
Now remote connections commands change as well and these have to pass
specific ssh connection port.
```sh
$ ssh root@xxx.xxx.xxx.xxx -p 2200 -i ~/.ssh/udacity_key.rsa
```

```sh
$ ssh grader@xxx.xxx.xxx.xxx -p 2200 -i ~/.ssh/fullstack
```

Installing NTP
-----------------

To install ntpd, from a terminal prompt enter:

```sh
$ sudo apt-get install ntp
```

Edit /etc/ntp.conf to add/remove server lines. By default these servers are configured:

```sh
# Use servers from the NTP Pool Project. Approved by Ubuntu Technical Board
# on 2011-02-08 (LP: #104525). See http://www.pool.ntp.org/join.html for
# more information.
server 0.ubuntu.pool.ntp.org
server 1.ubuntu.pool.ntp.org
server 2.ubuntu.pool.ntp.org
server 3.ubuntu.pool.ntp.org
```

Some near NTP server URL can be found here:
`http://www.pool.ntp.org/en/`

After changing the config file you have to reload the ntpd:

```sh
sudo service ntp reload
```

Use ntpq to see more info:

```sh
# sudo ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
+stratum2-2.NTP. 129.70.130.70    2 u    5   64  377   68.461  -44.274 110.334
+ntp2.m-online.n 212.18.1.106     2 u    5   64  377   54.629  -27.318  78.882
*145.253.66.170  .DCFa.           1 u   10   64  377   83.607  -30.159  68.343
+stratum2-3.NTP. 129.70.130.70    2 u    5   64  357   68.795  -68.168 104.612
+europium.canoni 193.79.237.14    2 u   63   64  337   81.534  -67.968  92.792
```
look at `https://help.ubuntu.com/lts/serverguide/NTP.html`
 for more information


Configuring the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
-----------------------------------------------------------------------------------
```sh
$ sudo ufw status
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow ssh
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 123/udp
$ sudo ufw allow www
$ sudo ufw enable
$ sudo ufw status
```

Installing Apache
-------------------

```sh
$ sudo apt-get install apache2
```

Setting Up Apache, mod_wsgi and Flask
--------------------------------------
 These lines are based on guide "How To Deploy a Flask Application on an Ubuntu VPS" available here: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
 
 #### Install and Enable mod_wsgi
 ```sh
 $ sudo apt-get install libapache2-mod-wsgi python-dev
 $ sudo a2enmod wsgi 
 ```
 It is possible to enable apache modules using the sudo a2enmod command, and disable them using sudo a2dismod
 
 #### Creating a Flask App
 The Project that is going to be served is called catalog. This way, this is the
 name used in the set of commands writen below. Use the following command to move to the /var/www directory:
 ```sh
 $ cd /var/www 
 ```
 
 Create the application directory structure using mkdir as shown. 
 ```sh
 $ sudo mkdir catalog
 ```
 Move inside this directory using the following command:
 ```sh
 $ cd catalog
 ```
Create a file called xsvr.py inside just created catalog directory
 ```sh
 $ sudo nano xsvr.py
 ```

Add next content to this file
 ```sh
 from catalog import app

if __name__ == '__main__':
    app.run()
 ```

 Create another directory catalog by giving following command:
 ```sh
 $ sudo mkdir catalog
 ```
 Then, move inside this directory and create two subdirectories named static and templates using the following commands:
 ```sh
$ cd catalog
$ sudo mkdir static templates
 ```
 
The directory structure should now look like this:
```sh
|----catalog
|---------xsvr.py
|---------catalog
|--------------static
|--------------templates
```

Now, create the __init__.py file that will contain the flask application logic.
```sh
$ sudo nano __init__.py
Add following logic to the file:

from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, fullstack Udacity"
```
Save and close the file.

#### Install Flask
##### virtualenv
https://pypi.python.org/pypi/virtualenv

``virtualenv`` is a tool to create isolated Python environments.

The basic problem being addressed is one of dependencies and versions,
and indirectly permissions. Imagine you have an application that
needs version 1 of LibFoo, but another application requires version 2. How can you use both these applications? If you install everything into ``/usr/lib/python2.7/site-packages`` (or whatever your
platform's standard location is), it's easy to end up in a situation
where you unintentionally upgrade an application that shouldn't be
upgraded.

Setting up a virtual environment will keep the application and its dependencies isolated from the main system. Changes to it will not affect the cloud server's system configurations.

In this step, we will create a virtual environment for catalog application.

We will use pip to install virtualenv and Flask. If pip is not installed, install it on Ubuntu through apt-get.
```sh
$ sudo apt-get install python-pip 
```

If virtualenv is not installed, use pip to install it using following command:
```sh
$ sudo pip install virtualenv 
```

Give the following command (where `venv` is the name  that will be given to the temporary environment for the catalog project):
```sh
$ sudo virtualenv venv
```

Now, install Flask in that environment by activating the virtual environment with the following command:
```sh
$ source venv/bin/activate 
```
Give this command to install Flask inside:
```sh
$ sudo pip install Flask
```
Next, run the following command to test if the installation is successful and the app is running:
```sh
$ sudo python xsvr.py 
```
It should display "Running on http://localhost:5000/" or "Running on http://127.0.0.1:5000/". If you see this message, you have successfully configured the app. Stop server with Ctl+C and deactivate the environment, give the following command:
```sh
$ deactivate
```
 #### Configure and Enable a New Virtual Host
 
 Issue the following command in your terminal:
 ```sh
 $ sudo nano /etc/apache2/sites-available/catalog.conf
 ```
 
 Add the following lines of code to the file to configure the virtual host. Be sure to change the ServerName to your domain or cloud server's IP address:
 
 ```sh
 <VirtualHost *:80>
		ServerName xxx.xxx.xxx.xxx
		ServerAdmin asintota@gmail.com
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
 Save and close the file.

Enable the virtual host with the following command:

```sh
$ sudo a2ensite catalog
```
It is possile to enable apache sites you've set up using sudo a2ensite, and disable them using sudo a2dissite.

#### Creating the .wsgi File

Apache will a the .wsgi file to serve the Flask catalog app. Move to the `/var/www/catalog` directory and create a file named catalog.wsgi with following commands:

```sh
$ cd /var/www/catalog
$ sudo nano catalog.wsgi 
```

Add the following lines of code to the catalog.wsgi file:

```sh
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'catalogkey'
```

Now catalog directory structure should look like this:
```sh
|--------catalog
|----------------xsvr.py
|----------------catalog
|-----------------------static
|-----------------------templates
|-----------------------venv
|-----------------------__init__.py
|----------------flaskapp.wsgi
```

#### Restart Apache

Restart Apache with the following command to apply the changes:

```sh
$ sudo service apache2 restart 
```

You may see a message similar to the following:

```sh
Could not reliably determine the VPS's fully qualified domain name, using 127.0.0.1 for ServerName 
```


This message is just a warning, and you will be able to access your virtual host without any further issues. To view your application, open your browser and navigate to the domain name or IP address that you entered in your virtual host configuration.


*note: tail /var/log/apache2/error.log is your homie


Getting python installed package list
--------------------------------------
Sometimes is usefull to findout what python packages are
installed and what version these are. For doing this type:
```sh
$ pip freeze
```
This the list output for my first commited catalog project vagrant VM:
```sh
Cheetah==2.4.4
Flask==0.9
Flask-Login==0.1.3
Flask-SeaSurf==0.2.1
Jinja2==2.7.2
Landscape-Client==14.12
MarkupSafe==0.18
PAM==0.4.2
PyYAML==3.10
SQLAlchemy==0.8.4
SecretStorage==2.0.0
Twisted-Core==13.2.0
Twisted-Names==13.2.0
Twisted-Web==13.2.0
WTForms==2.0.2
Werkzeug==0.8.3
apt-xapian-index==0.45
argparse==1.2.1
bleach==1.4.1
blinker==1.3
chardet==2.0.1
cloud-init==0.7.5
colorama==0.2.5
configobj==4.7.2
dicttoxml==1.6.6
html5lib==0.999
httplib2==0.9.1
itsdangerous==0.22
jsonpatch==1.3
jsonpointer==1.0
keyring==3.5
launchpadlib==1.10.2
lazr.restfulclient==0.13.3
lazr.uri==1.0.3
oauth==1.0.1
oauth2client==1.4.12
prettytable==0.7.2
psycopg2==2.4.5
pyOpenSSL==0.13
pyasn1==0.1.8
pyasn1-modules==0.0.6
pycrypto==2.6.1
pycurl==7.19.3
pygobject==3.12.0
pyinotify==0.9.4
pyserial==2.6
python-apt==0.9.3.5ubuntu1
python-debian==0.1.21-nmu2ubuntu2
requests==2.2.1
rsa==3.2
simplejson==3.3.1
six==1.9.0
ssh-import-id==3.21
urllib3==1.7.1
wadllib==1.3.2
wheel==0.24.0
wsgiref==0.1.2
zope.interface==4.0.5
```
Installing PostgreSQL
---------------------
```sh
$ sudo apt-get install postgresql
```


Intalling more required packages
---------------------------------
Going back to `/var/www/catalog/catalog`, virtual environment
is actavated again
```sh
source venv/bin/activate
```
### Psycopg
http://initd.org/psycopg/

Psycopg is the most popular PostgreSQL adapter for the Python programming language. At its core it fully implements the Python DB API 2.0 specifications. Several extensions allow access to many of the features offered by PostgreSQL.
```sh
$ sudo apt-get install python-psycopg2
```

### Flask-SeaSurf
https://flask-seasurf.readthedocs.org/en/latest/

SeaSurf is a Flask extension for preventing cross-site request forgery (CSRF).
```sh
$ sudo pip install flask-seasurf
```

### dicttoxml
https://pypi.python.org/pypi/dicttoxml

Converts a Python dictionary or other native data type into a valid XML string.
```sh
$ sudo pip install dicttoxml
```
### Flask-Login
https://pypi.python.org/pypi/Flask-Login

Flask-Login provides user session management for Flask. It handles the common tasks of logging in, logging out, and remembering your users’ sessions over extended periods of time.
```sh
$ sudo pip install Flask-Login==0.1.3
```

### SQLAlchemy
```sh
$ sudo pip install SQLAlchemy==0.8.4
```

### oauth2client
```sh
$  sudo pip install oauth2client==1.4.12
```

### WTForms
https://pypi.python.org/pypi/WTForms

```sh
$ sudo pip install WTForms==2.0.2
```

and
```sh
$ deactivate
```


Creating grader postgres user(role)
-------------------------------------
Initiate a postgres session:
```sh
$ sudo -i -u postgres
```
Then initiate an interactive user creation with password as required:
```sh
postgres$ createuser --interactive -P grader
Enter password for new role: 
Enter it again: 
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) y
Shall the new role be allowed to create more new roles? (y/n) n
```
#### Some postgres commands
##### Drop a user
```sh
postgres$ dropuser grader
```
##### current role list
```sh
postgres$ psql
postgres=# SELECT rolname FROM pg_roles;
```
##### delete a data base
```sh
postgres$ psql
postgres=# DROP DATABASE catalog;
```

grader creates a database for catalog project
----------------------------------------------
Get out from postgres session and type:
```sh
$ createdb -U grader --locale=en_US.utf-8 -E utf-8 -O grader catalog -T template0
```

Update database path when engine is created
------------------------------------------
When SQlite was used engine object in python code was built with:
```sh
engine = create_engine('sqlite:///catalog.db')
```
Now this code must be replaced by:
```sh
engine = create_engine('postgresql://grader:grader@localhost/catalog')
```

see more in
`http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html`


Copying static and templates content from development machinne to VPS
-----------------------------------------------------------------------
```sh
$ scp -P 2200 -i ~/.ssh/udacity_key.rsa -r static root@52.35.199.115:/var/www/catalog/catalog

$ scp -P 2200 -i ~/.ssh/udacity_key.rsa -r templates root@52.35.199.115:/var/www/catalog/catalog
```



web resources permissions
---------------------------
It is important to guarantee reading permissions for other users to web resources.
This includes javascript, style and images files and some other
```sh
-rw-r----- 1 root root 283668 Jan 13 04:08 521093__the-grasshopper_p.jpg
-rw-r----- 1 root root 129027 Jan 13 04:08 bootstrap.min.css
-rw-r--r-- 1 root root  36816 Jan 13 04:09 bootstrap.min.js
drwxr-xr-x 2 root root   4096 Jan 13 04:09 css
drwxr-xr-x 2 root root   4096 Jan 13 04:09 fonts
-rw-r----- 1 root root   8865 Jan 13 04:09 images.jpeg
-rw-r----- 1 root root  95957 Jan 13 04:09 jquery-1.11.3.min.js
drwxr-xr-x 2 root root   4096 Jan 13 04:09 js
-rw-r----- 1 root root  16676 Jan 13 04:09 linux-embedded.jpg

grader@ip-10-20-44-163:/var/www/catalog/catalog/static$ sudo chmod 644 bootstrap.min.css bootstrap.min.js jquery-1.11.3.min.js linux-embedded.jpg

-rw-r----- 1 root root 283668 Jan 13 04:08 521093__the-grasshopper_p.jpg
-rw-r--r-- 1 root root 129027 Jan 13 04:08 bootstrap.min.css
-rw-r--r-- 1 root root  36816 Jan 13 04:09 bootstrap.min.js
drwxr-xr-x 2 root root   4096 Jan 13 04:09 css
drwxr-xr-x 2 root root   4096 Jan 13 04:09 fonts
-rw-r----- 1 root root   8865 Jan 13 04:09 images.jpeg
-rw-r--r-- 1 root root  95957 Jan 13 04:09 jquery-1.11.3.min.js
drwxr-xr-x 2 root root   4096 Jan 13 04:09 js
-rw-r--r-- 1 root root  16676 Jan 13 04:09 linux-embedded.jpg

```
Otherwise server can respond with code `403`


Some Denial of Service DoS Protection
---------------------------------------

Issue the command
```sh
$ sudo apt-get -y install libapache2-mod-evasive
```
Issue the command 
```sh
sudo mkdir -p /var/log/apache2/evasive
```
Issue the command 
```sh
sudo chown -R www-data:root /var/log/apache2/evasive
```
Open the `/etc/apache2/mods-available/mod-evasive.load` file (using sudo and your favorite text editor) and append the following to the bottom of that file (this is one configuration per line):
```sh
DOSHashTableSize 2048
DOSPageCount 20
DOSSiteCount 300
DOSPageInterval 1.0
DOSSiteInterval 1.0
DOSBlockingPeriod 10.0
DOSLogDir "/var/log/apache2/evasive"
```

This same cofiguration with comments
```sh
DOSHashTableSize 2048
DOSPageCount 20  # maximum number of requests for the same page
DOSSiteCount 300  # total number of requests for any object by the same client IP on the same listener
DOSPageInterval 1.0 # interval for the page count threshold
DOSSiteInterval 1.0  # interval for the site count threshold
DOSBlockingPeriod 10.0 # time that a client IP will be blocked for
DOSLogDir "/var/log/apache2/evasive" # log directory
```
Save the file and restart Apache
For more web protections modules look at :
`http://www.techrepublic.com/blog/smb-technologist/secure-your-apache-server-from-ddos-slowloris-and-dns-injection-attacks/`
 and
 `https://httpd.apache.org/docs/trunk/misc/security_tips.html`




Disable root SSH Login
------------------------
Edit `/sshd_config`, change `PermitRootLogin` configuration option to `PermitRootLogin   no`
```sh
$ sudo  nano /etc/ssh/sshd_config
...
$ sudo service ssh restart

```
