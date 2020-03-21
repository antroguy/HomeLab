
Home Lab Setup Setting up a Guacamole server: Requirements: Ubuntu 18.04 most recent version or sooner. Really doesn’t matter since we are using dockers.

Install docker with command “sudo apt install docker.io”

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
