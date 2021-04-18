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

```c
# docker-compose.yml 

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


 Run the command to bring up all the 3 server\
 $ docker-compose up -d         /* Why -d: it is detached mode to run the container without killing itself */
 
 
 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Now, all the 3 server are up and running. However we need to install below two server in "remote_user" in order to take back up from the db_host and to upload the file into the aws s3.
1. mysql     ( to take back-up from db server)
2. aws cli   ( to upload the file into the AWS S3)

So, let's intall them. :-)

We can nstall them in two ways:
1. Go inside the  remote-host container and run install command:
* $ docker exec -it <container_name pr id> bash
* $ yum install -y mysql   /* yum for censtos and apt for ubuntu: this command will install the mysql client
2. We can add this installatin process in the docker file itself as below:
```c
# Just above the CMD command in the Dockerfile ( remote-user) , add this line to install the mysql client

RUN yum install -y mysql
CMD /usr/sbin/sshd -D
```
Installation of AWS cli:
1. inside the container command line
```c
RUN curl "https://awscli.amazonaws.com/   awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    yum install -y unzip && \
    unzip awscliv2.zip && \
    ./aws/install
  ```
  
2. modify in the Dockerfile.

```c  
# Just above the CMD command in the Dockerfile ( remote-user) , add this line to install the mysql client

RUN curl "https://awscli.amazonaws.com/   awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    yum install -y unzip && \
    unzip awscliv2.zip && \
    ./aws/install
CMD /usr/sbin/sshd -D
```
=========================================================================================================================

Now let's get into the db-server (mysql) from remote user and create a database and populate with values
* Login to the remote-host container
$ docker exec -id <container_name or id> bash
* from there login to the db-server
$ mysqsl -u root -h db_host -p
* there will be an passwd prompt: Enter the passwd which we have set earlier. <1234> in our case

Then run the below code to create the databse and pipulate it.

```c
# To create a table in mysql
$ create database testdb;

# To use the database
$ use testdb;

# To create tables
$ create table info (name varchar(20), lastname varchar(20), age int(10));

# To describe the table
$desc info;

# To insert the data into the table
$ insert into info values ('sam', 'sanasam', 32);

$ select * from info



```


============================================================
#### Now let's work on AWS console 
we need to create a S3 bucket and a IAM user from the AWS console manually. I hope this can be done easily.
#### Creating S3 bucket
* login to the aws console
* go to the S3 service
* click on create bottom
* give the name <db_data_bucket> in our case

#### Creating IAM user
* login to the AWS console
* go to the IMA service
* click on user
* click on add
* give the name <remote_user>
* click on programatic
* click on next
* attach the policy:
* adminastrationaccess ( full access)

After creating the IAM user , download the acess key for aws cli configuration:

- - - - - - - - - - - - - - - - - - - - - - - - 
Lets configure the AWS cli
Login to the remote-host and follow the below commands:
```
$ aws configure
# provide the acess key
# provide the secrete key
Enter
# providr the output as <json>

# after the completion of configuarution, jsut confirm that we have configure correctly or not y running the below command:
$ aws s3 list

```
+++++++++++++++++++++++++++++++++++++++++
### Let's take up the backup form db_host
In order to take up the backup, we have to run the below command:
```c
$ mysqldump -u root -h db_host -p testdb>/tmp/db_data.sql

```
the above code will take the bac up from the db_host server and put it inside the /tmp folder in the remote-host server.

##### Lests upload the backup into the AWS s3 buscket

```c
$ aws s3 copy /tmp/db_data.sql s3://db_data_bucket/db_data.sql

## aws s3 <file path with file name> s3://<bucket_name/<file name>

```
## BOOM!! the backup is upload ed in the AWS S3 bucket.







