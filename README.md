# HomeLab
Home Lab Setup
Setting up a Guacamole server:
Requirements: Ubuntu 18.04 most recent version or sooner. Really doesn’t matter since we are using dockers.
1.	Install docker with command “sudo apt install docker.io”
2.	pull guacamole guacd docker with “sudo docker pull guacamole/guacd”
3.	pull guacamole docker with “sudo docker pull guacamole/guacamole”
4.	grab mysql database structure with
a.	“docker run –rm guacamole/guacamole /opt/guacamole/bin/initdb.sh   --mysql > initdb.sql
5.	Set up guacamole guacd with “sudo docker run -d –name guacdDocker guacamole/guacd
6.	Pull guacamole mysql docker with “sudo docker pull mysql
7.	Create docker container and run it using “sudo docker run -d –name=sqlDocker –env=”MYSQL_ROOT_PASSWORD=xxxxxxxx” mysql”
8.	Download mysql-client with “sudo apt install mysql-client
9.	Inspect mysql for ip address with “sudo docker inspect sqlDocker”
10.	Connect to mysql server using “mysql -u root -p -h 172.15.0.2”, type password when prompted
11.	Create guacamole db in sql cli “create database guacamole_db
12.	If you type “show databases”, you should see guacamole_db
13.	mysql -u root -p -h 172.17.0.2 guacamole_db < initdb.sql
14.	log back into sql server and type in “use guacamole_db” to access the database.
15.	Create a user for the guacamole server “create user ‘guacadmin’@’localhost’ identified by ‘password’;
16.	Grant all privileges on guacamole_db to ‘guacadmin’@’localhost’;
17.	Type in “select host, user from mysql.user;” to verify the user guacadmin as been created.
18.	Type in “show grants for ‘guacadmin’@’localhost’; “ to verify all privileges for guaccadmin_db have been granted to the guacadmin user
