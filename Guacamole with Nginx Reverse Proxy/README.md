
**Home Lab - Setting up Guacamole Server**
The purpose of this guide is to provide instructional value on how to deploy a Guacamole server as a jump server for a home network.
This guide will provide instrucction on how to deploy a guacamole server and mysql server using docker instances, how to set up SSL encryption for the guac server using nginx as a reverse proxy, and how to generate an SSL certificate through letsencrypt.

**Requirements:**
* Ubuntu 18.04 most recent version or sooner. The version is not critical since I will be using dockers to deploy both my guacamole and mysql server.
  * Ubuntu 18.04 ISO can be downloaded from http://releases.ubuntu.com/18.04.4/. I will be using ESXI to host my ubuntu server.
* Ability to NAT/port forward services from your router (I am using pfsense as my firewall)
* Ownership of a domain (I purchased my domain through google domains)

1. On your Ubuntu server install the docker application.
```python
s = “sudo apt install docker.io”
print s
```
pull guacamole guacd docker with “sudo docker pull guacamole/guacd”

pull guacamole docker with “sudo docker pull guacamole/guacamole”

grab mysql database structure with a. “docker run –rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --mysql > initdb.sql

Set up guacamole guacd with “sudo docker run -d –name guacdDocker guacamole/guacd
Pull guacamole mysql docker with “sudo docker pull mysql
Create docker container and run it using “sudo docker run -d –name=sqlDocker –env=”MYSQL_ROOT_PASSWORD=xxxxxxxx” mysql”
Download mysql-client with “sudo apt install mysql-client
Inspect mysql for ip address with “sudo docker inspect sqlDocker”
Connect to mysql server using “mysql -u root -p -h 172.15.0.2”, type password when prompted
Create guacamole db in sql cli “create database guacamole_db
If you type “show databases”, you should see guacamole_db
mysql -u root -p -h 172.17.0.2 guacamole_db < initdb.sql
log back into sql server and type in “use guacamole_db” to access the database.
Create a user for the guacamole server “create user ‘guacadmin’@’%’ identified by ‘password’; a. If I used localhost instead of ‘%’ it didn’t work
“Grant all privileges on guacamole_db.* to ‘guacadmin’@’localhost’;”
Type in “select host, user from mysql.user;” to verify the user guacadmin as been created.
Type in “show grants for ‘guacadmin’@’localhost’; “ to verify all privileges for guaccadmin_db have been granted to the guacadmin user
