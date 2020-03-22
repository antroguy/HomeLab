
**Home Lab - Setting up Guacamole Server**
The purpose of this guide is to provide instructional value on how to deploy a Guacamole server as a jump server for a home network.
This guide will provide instrucction on how to deploy a guacamole server and mysql server using docker instances, how to set up SSL encryption for the guac server using nginx as a reverse proxy, and how to generate an SSL certificate through letsencrypt.

**Requirements:**
* Ubuntu 18.04 most recent version or sooner. The version is not critical since I will be using dockers to deploy both my guacamole and mysql server.
  * Ubuntu 18.04 ISO can be downloaded from http://releases.ubuntu.com/18.04.4/. I will be using ESXI to host my ubuntu server.
* Ability to NAT/port forward services from your router (I am using pfsense as my firewall)
* Ownership of a domain (I purchased my domain through google domains)

1. On your Ubuntu server install the docker tool.
```
$ sudo apt install docker.io”
```
2. Pull the guacamole/guacd, guacamole/guacamole, and mysql docker containers.
```
$ sudo docker pull guacamole/guacd
```
```
$ sudo docker pull guacamole/guacamole
```
```
$ sudo docker pull mysql
```

3. Now we will set up the guacamole/guacd container, this will provide the guacamole guacd daemon that will accept and manage user connections. Run the following command, where _guacd-name_ is the name you will assign to your guacamole/guacd container. The _-d_ argument runs the docker container in the background.
```
$ sudo docker run -d –name guacd-name guacamole/guacd
```
4. Next we will set up the mysql container. First create an sql database structure for the guacamole database. Fortunately there is a initdb.sh script that will create the guacamole database structure. 
```
$ docker run –rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```
5. Start the mysql container, where _sql-name_ is the name you will assign the sql container, and _mypassword_ is the root password for mysql.
```
sudo docker run -d –name=sql-name –env=”MYSQL_ROOT_PASSWORD=mypassword” mysql
```
6. Download mysql-client in order to connect to the mysql database.
```
sudo apt install mysql-client
```
7. Inspect the mysql container for the containers ip address. 
```
sudo docker inspect sqlDocker
```

Connect to mysql server using “mysql -u root -p -h 172.15.0.2”, type password when prompted
Create guacamole db in sql cli “create database guacamole_db
If you type “show databases”, you should see guacamole_db
mysql -u root -p -h 172.17.0.2 guacamole_db < initdb.sql
log back into sql server and type in “use guacamole_db” to access the database.
Create a user for the guacamole server “create user ‘guacadmin’@’%’ identified by ‘password’; a. If I used localhost instead of ‘%’ it didn’t work
“Grant all privileges on guacamole_db.* to ‘guacadmin’@’localhost’;”
Type in “select host, user from mysql.user;” to verify the user guacadmin as been created.
Type in “show grants for ‘guacadmin’@’localhost’; “ to verify all privileges for guaccadmin_db have been granted to the guacadmin user
