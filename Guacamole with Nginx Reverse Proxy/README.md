
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
