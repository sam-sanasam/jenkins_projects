### Here we are going to take the backup from the db server (mysql) and upload to the aws s3 


We have below set up:
- jenkins as jenkins server
- db-host as mysql server
- remote-host as remote user


The description of the project is ngiven below:\
<img width="350" alt="jenkins+db+aws+s3" src="https://user-images.githubusercontent.com/68118215/115134523-7e218d80-a02e-11eb-8fad-aee900da25a5.png">

The remote user from remote host will take back up from the db-server which is running mysql and uload the back-up to AWS S3.\
This can be done in two ways:
1. Manual approch
2. Automatic approach


#### Let's Get Started

##### We will create a mysql container using dcoker-compose.

```
version= '3'
services:
   jenkins:
     container_name: jenkins
     images: jankins/jenkins
     ports:
       - "8080:8080"
     volumes:
       - "$PWD/jenkins_home:/var/jenkins_home"
     networks:
       - net
   remote_host:
     container_name: remote-host
     image: remote-host
     build:
       context: centos7    /* this is the path where docker file reside*/
     networks:
       - net 
     
    db_host:
      container_name: db
      image: mysql:5.7
      environment:
        - "MYSQL_ROOT_PASSWORD=1234"   /* in order to give an password for the db server use the 'Envaironment variable ' to create the passwd */
     volumes:
        - "$PWD/db_data:/var/lib/mysql"   /* To make persistence the data, we have to create a volume in our host system . Here db_data directory is created under PWD path and the data store in the mysql server under /var/lib/mysql will copy into the 'db_data'
     
     networks:
       - net
    
networks:
   net:
   
   
   
```
