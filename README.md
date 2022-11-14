#****Docker House cleaning up
#Kill All Running Containers
docker kill $(docker ps -q)
#Delete All Stopped Docker Containers
docker rm $(docker ps -a -q)
#Remove a Docker Image 
docker rmi <image name>
#Delete All Docker Images
docker rmi $(docker images -q)
#Delete All Untagged (dangling) Docker Images
docker rmi $(docker images -q -f dangling=true)
#Remove Dangling Volumes (after volume is not associated with a container, it is considered 'dangling'
docker volume rm -f $(docker volume ls -f dangling=true -q)


#In order to run docker on Windows (1), you have to 
1) Have WSL 2 installed (documentation below also links to WSL 2 installation)
2) Install Docker desktop (make sure that background is WSL 2): https://docs.docker.com/desktop/install/windows-install/
3) Run commands in "Ubuntu" windows

#run mongo (no persistnat data)
docker run -p 27017:27017 -d mongo

#run mongo (persistant data). Note: we have previously created the docker/dockerdata/mongo directories
/home/carolijime/docker/dockerdata/mongo
docker run -p 27017:27017 -v /home/carolijime/docker/dockerdata/mongo:/data/db -d mongo

#run rabbitmq management (mapping different ports)
docker run -d --hostname guru-rabbit --name some-rabbit -p 8080:15672 -p 5671:5671 -p 5672:5672 rabbitmq:3-management

#run MySQL image, using port 3306. Persistant storage. User root and password null. Use Enrironment variables to accomplish this (create dockerdata/mysqlTEMP directory in advance)
docker run --name guru-mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -v /home/carolijime/docker/dockerdata/mysqlTEMP:/var/lib/mysql -p 3306:3306  -d mysql
--***NOTE**** command above will run only if local (windows) mysql is stopped, use command below if you have local mysql installed and port 3306 is already in use
docker run --name guru-mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -v /home/carolijime/docker/dockerdata/mysqlTEMP:/var/lib/mysql -p 3307:3306 -d mysql

--***NOTE**** I was unable to connect (using MySQL WorkBench app), but tried different solutions, listed below
**1) setting a password for root
docker run --name guru-mysql -e MYSQL_ROOT_PASSWORD=abc -v /home/carolijime/docker/dockerdata/mysqlTEMP:/var/lib/mysql -p 3307:3306 -d mysql 

**2) 
a) Restarted computer
b) stopped local mysql.exe tasks (task manager)
c) run below command
	docker run --name guru-mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -v /home/carolijime/docker/dockerdata/mysqlTEMP:/var/lib/mysql -p 3306:3306  -d mysql
d) try connection (port 3306) in MySQL WorkBench - Worked
e) stop just created container
f) run below command
	docker run --name guru-mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -v /home/carolijime/docker/dockerdata/mysqlTEMP:/var/lib/mysql -p 3307:3306 -d mysql
g) try connection (port 3307) in MySQL WorkBench - Worked
h) restart computer
	
**3) configuring root's host
mysql 8 requires some configurations to make it work.

Run:

docker exec -it <mysql_container_id> bash 

Then execute this:

mysql -u root -p mysql

Then  run this:



GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
use mysql;
UPDATE mysql.user SET host='%' WHERE user='root';
 
Type Ctrl+D and restart mysql with the following command:

mysqld restart

*******************************************************
