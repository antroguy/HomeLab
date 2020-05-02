
# Home Lab - Setting up Guacamole Server
The purpose of this guide is to provide instructional value on how to deploy a Guacamole server as a jump server for a home network.
This guide will provide instruction on how to deploy a guacamole server and mysql server using docker instances, and how to set up https for the guac server using nginx as a reverse proxy.

**Requirements:**
* Ubuntu 18.04 most recent version or sooner. The version is not critical since I will be using dockers to deploy both my guacamole and mysql server.
  * Ubuntu 18.04 ISO can be downloaded from http://releases.ubuntu.com/18.04.4/. I will be using ESXI to host my ubuntu server.
* Ability to NAT/port forward services from your router (I am using pfsense as my firewall)
* Ownership of a domain (I purchased my domain through google domains)

### STEP 1: Setting up Guacamole server using docker containers
On your Ubuntu server install the docker tool.
```
$ sudo apt install docker.io”
```
Pull the guacamole/guacd, guacamole/guacamole, and mysql docker containers.
```
$ sudo docker pull guacamole/guacd
```
```
$ sudo docker pull guacamole/guacamole
```
```
$ sudo docker pull mysql
```

Now we will set up the guacamole/guacd container, this will provide the guacamole guacd daemon that will accept and manage user connections. (NOTE: _guacd-name_ is the name you will assign to your guacamole/guacd container. The _-d_ argument runs the docker container in the background)
```
$ sudo docker run -d –name guacd-name guacamole/guacd
```
Next we will set up the mysql container. First create an sql database structure for the guacamole database. Fortunately, there is a initdb.sh script within the guacamole container that will create the guacamole database structure. The following command will run the initdb.sh script within the guacamole container, output the dataase structure to initdb.sql, and then remove the docker container. 
```
$ docker run –rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```
Start the mysql container (NOTE: _sql-name_ is the name you will assign the sql container, and _mypassword_ is the root password for mysql)
```
$ sudo docker run -d –name=sql-name –env=”MYSQL_ROOT_PASSWORD=mypassword” mysql
```
Download mysql-client in order to connect to the mysql database.
```
$ sudo apt install mysql-client
```
Inspect the mysql container, and take note of the ip address. 
```
$ sudo docker inspect sql-name
```
![alt text](https://github.com/antroguy/HomeLab/blob/master/Guacamole%20with%20Nginx%20Reverse%20Proxy/Images/mysqlNetwork_inspect.PNG)
Connect to the mysql server using the IP address identified in the previous step.
```
$ mysql -u root -p -h 172.15.0.2
```
Once connected to the mysql server, create the guacamole database.
```
$ create database guacamole_db
```
If you type “show databases”  you should see the guacamole database created. Exit the mysql server. 
Next  we have to set up the guacamole_db structure. We ill import the initdb.sql file that we created previously into the guacamole_db. 
```
$ mysql -u root -p -h 172.17.0.2 guacamole_db < initdb.sql
```
Log back into sql server via mysql-client and type in “use guacamole_db” to access the guacamole database. Create a user for the guacamole server using the following command. (NOTE: _guacadmin_ is the username you want to create for the guacamole service, and _password_ is the password you want to assign to the user. The _%_  is used to allow the guacamole account to login from any host, the field can changed to localhost or a static ip address instead.)
```
$ create user ‘guacadmin’@’%’ identified by ‘password’; 
```
Next, grant all privledges to the guacamole account to the guacamole database.
```
$ Grant all privileges on guacamole_db.* to ‘guacadmin’@’localhost’;
```
You can type in the following commands to verify the guacamole service account has been created and granted the appropriate privledges.
```
$ select host, user from mysql.user;” 
```
```
$ show grants for ‘guacadmin’@’localhost’; 
```
Exit out of your mysql server command line interface. Set up the guacamole/guacamole container and link the mysql container, guacd containter, and guacamole database to their corresponding services
```
$ docker run --name guacamoleDocker 
  --link guacd-name:guacd 
  --link sql-name:mysql 
  -e MYSQL_DATABASE=guacamole_db 
  -e MYSQL_USER= 
  -e MYSQL_PASSWORD=mypassword 
  -d -p 8080:8080 guacamole/guacamole
```
You should now be able to access your guacamole web application by opening a web browser and going to http://localhost:8080/guacamole/.

### STEP 2: Setting up HTTPS using Nginx
Before setting up your Nginx server you will need to set up your publicly accessible domain name. I purchased a domain through google domains at https://domains.google.com. Once you purchase your domain you will need to create an 'A' record that points to your public IP address. (For this writeup, our 'A' record Name will be "guacamoleServer")

![alt text](https://github.com/antroguy/HomeLab/blob/master/Guacamole%20with%20Nginx%20Reverse%20Proxy/Images/googleDomain.PNG)

Next you will need to create a NAT rule on your firewall/router to port forward all http and https traffic to your guacamole servers private IP address. I am using Pfsense as my firewall. From pfsense you will need to go to Firewall -> NAT to create youre NAT rule. 

![alt text](https://github.com/antroguy/HomeLab/blob/master/Guacamole%20with%20Nginx%20Reverse%20Proxy/Images/pfSenseNAT.png)

You will also need to create a firewall rule to allow http and https traffic inbound to your guacamoles internal IP address.

![alt text](https://github.com/antroguy/HomeLab/blob/master/Guacamole%20with%20Nginx%20Reverse%20Proxy/Images/pfsenseRule.PNG)

Now it is time to setup SSL over HTTPS using Nginx as a reverse proxy for our guacamole server. First you will need to install nginx. Run the following command:
```
$ Sudo apt install nginx
```
Next go to the directory /etc/nginx/sites-available/ and make a copy of the default configuration file, naming it something like guacamoleProxy (For identification purposes)
```
$ Sudo cp default guacamoleProxy
```
Edit the guacamoleProxy config file. You will need to initially set it up to act as a reverse proxy for HTTP traffic over port 80. You can use my default file as an example. (NOTE: _server_name_ is your domain name with the 'A' record that was created previously)

![alt text](https://github.com/antroguy/HomeLab/blob/master/Guacamole%20with%20Nginx%20Reverse%20Proxy/Images/nginxConfig.PNG)

Go into the directory /etc/nginx/sites-enabled/ and create a symbolic link to guacamoleProxy configuration file you just created.

```
ln -s /etc/nginx/sites-available/guacamoleProxy /etc/nginx/sites-enabled/guacamoleProxy
```
If you browse to http://localhost/ from your web browse it should redirect to your guacamole server. (Instead of browsing to http://localhost:8080/guacamole/#/)

Next we will setup HTTPS for our nginx server, and have it redirect all traffic over port 443 to our guacamole server. We will use LetsEncrypt to generate a free SSL certificate. First install certbot for nginx on your guac host. 
```
$ Sudo apt-get install python-certbot-nginx
```
Run certbot to generate your SSL certificate and configure HTTPS for your nginx server. 
```
$ Sudo certbot --nginx
```
Follow the prompt as directed. Once complete you should be able to access your guacamole server via HTTPS through your domain name (e.g. https://guacamoleserver.domain.com)
