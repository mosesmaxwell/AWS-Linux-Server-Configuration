# AWS-Linux-Server-Configuration
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.


## Configuration Steps

The IP address: 34.226.219.58

SSH port: 2200

The complete URL to hosted web application: http://34.226.219.58/

## Get your server

1. Start a new Ubuntu Linux server instance on Amazon Lightsail https://lightsail.aws.amazon.com/. Once you created the instance you will get the public IP, private IP and username. 
2. You can download your default private key from the Account page. It will be useful for login to SSH using putty.
3. Once the instance is running click the Connect using SSH button to launch the server using SSH.

## Secure your server

4. Run the following commands to update the installed packages.

```
$ sudo apt-get update
$ sudo apt-get upgrade

Additionally you can run below to upgrade systen packages not installed using apt-get:

$sudo apt-get dist-upgrade
```
Install finger
```
$ sudo apt-get install finger
```

5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw enable
```


6. Lightsail, click on Networking Change the SSH port from 22 to 2200.
```
Add port Custom TCP 123
Add port Custom TCP 2200
Remove port SSH TCP 22
```


7. Change the SSH port from 22 to 2200. Configure the Lightsail firewall to allow it.
Open the file /etc/ssh/sshd_config
```
$ sudo vi /etc/ssh/sshd_config
```

8. Now change the following data and save
```
Port 2200
PermitRootLogin no
PasswordAuthentication no
```

9. Now restart the SSH service
```
$ sudo service ssh restart
```


## Give grader access
In order for your project to be reviewed, the grader needs to be able to log in to your server.

10. Ceate a new user account named grader. Give name and password details as grader.
```
$ sudo adduser grader
```

11. Give grader the permission to sudo.

```
$ sudo vi /etc/sudoers.d/grader
```

12. Add below line and save the file.
```
grader ALL=(ALL) NOPASSWD:ALL
```

13. Create an SSH key pair for grader using the ssh-keygen tool. Share the private key to Reviewer of Udacity team. 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): /home/ubuntu/.ssh/idrsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/idrsa
Your public key has been saved in /home/ubuntu/.ssh/idrsa.pub

14. Create and copy the keys for tracer account.

```
$ su --login grader
$ sudo mkdir .ssh
$ sudo touch .ssh/auth_keys
$ exit
$ cat .ssh/idrsa.pub (copy the ssh key)
$ su --login grader
$ sudo vi .ssh/auth_keys (paste the key here)
$ sudo chmod 700 .ssh
$ sudo chmod 644 .ssh/auth_keys
$ exit
```

## Prepare to deploy your project


15. Configure the local timezone to UTC
```
$ sudo dpkg-reconfigure tzdata
```

16. Choose the option 'None of the Above' and then select UTC.

17. Install and configure Apache to serve a Python mod_wsgi application.
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
```

18. Install PostgreSQL
```
$ sudo apt-get install postgresql
````

19. Create a new database user named catalog that has limited permissions to access db. postgres is the root user. Use this user to create db and user role then change it to catalog user.
* Do not allow remote connections
```
$ sudo adduser catalog
$ sudo -u postgres -i

postgres:~$ createuser catalog
postgres:~$ createdb catalog

postgres:~$ psql

postgres=# ALTER DATABASE catalog OWNER TO catalog;
postgres=# ALTER USER catalog WITH PASSWORD 'catalog';
postgres:~$ exit
```

## Deploy the Item Catalog project

20. Install git
```
$ sudo apt-get install git
```

21. Clone and setup your CatalogApp project from the Github repository created earlier in this Nanodegree program.
```
$ git clone https://github.com/mosesmaxwell/CatalogApp.git catalog
```

22. Replace the db engine in models.py, loaddata.py and catalogapp.py files
```
engine = create_engine('postgresql://catalog:catalog@localhost:5432/catalog')
```

23. Install all dependencies
```
$ sudo apt-get -y install python-pip
$ sudo pip install SQLAlchemy
$ sudo pip install psycopg2
$ sudo pip install flask
$ sudo pip install oauth2client
$ sudo pip install requests
$ sudo pip install sqlalchemy_utils
```

24. Run models.py file to create tables for your CatalogApp project
```
python models.py
```

25. Run loaddata.py
```
python loaddata.py
```

26. Modify the file /etc/apache2/sites-enabled/000-default.conf to add the following line (Right before the closing </VirtualHost>)
```
WSGIScriptAlias / /var/www/html/myapp.wsgi
```

27. Modify the file /var/www/html/myapp.wsgi to add the following content
```
#!/usr/bin/python
import sys
import os
import logging
logging.basicConfig(stream=sys.stderr)
sys.stdout = sys.stderr
sys.path.insert(0,"/home/ubuntu/catalog/")
os.chdir("/home/ubuntu/catalog/")
from catalogapp import app as application   
```

28. Restart the server
```
$ sudo apache2ctl restart
```

29. Change the client_secrets.json with this server public url. Change the Google and Facebook domain url for Oauth access.

30. CatalogApp will run in http://34.226.219.58

31. If there is any errors please check the apache log file by login using SSH private key explain in step 2.
