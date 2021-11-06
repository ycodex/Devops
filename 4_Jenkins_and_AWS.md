# Jenkins And AWS

## Create MYSQL on Docker
- Modify the docker-compose file and add another service of mysql, the file should somewhat look like this
```
version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
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
      context: centos7
    networks:
      - net
  db_host:
    container_name: db
    image: mysql:5.7
    environment:
      - "MYSQL_ROOT_PASSWORD=2310"
    volumes:
      - "$PWD/db_data:/var/lib/mysql"
    networks:
      - net
networks:
  net:

```
- Also modify the remote host Dockerfile to install mysql and aws cli, refer below
```

FROM centos

RUN yum -y install openssh-server

RUN useradd remote_user && \
    echo "remote_user:2310" | chpasswd  && \
    mkdir /home/remote_user/.ssh && \
    chmod 700 /home/remote_user/.ssh

COPY remote-key.pub /home/remote_user/.ssh/authorized_keys

RUN chown remote_user:remote_user -R /home/remote_user/.ssh/ && \
    chmod 600 /home/remote_user/.ssh/authorized_keys

RUN ssh-keygen -A

RUN yum -y install mysql

RUN curl -O https://bootstrap.pypa.io/get-pip.py
RUN yum install -y python3
RUN python3 get-pip.py
RUN pip3 install awscli --upgrade

CMD /usr/sbin/sshd -D

```
- Then get the service up and running
- Get the bash prompt of the mysql container
> use the command `docker exec -ti db bash
- Then try logging into mysql 
 >use command `mysql -u root -p` 
- Create a sample database and table in mysql server and add some sample data
- Then go to AWS and search for S3 and create a bucket
- Go to IAM > user > add user and attach policy of S3 policy and download the access info file
- Log into remote host and create a backup manually, check below for reference command
`mysqldump -u root -h db_host -p testdb > /tmp/db.sql`

## Configure AWS cli and backup
Check the link for configurations
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html

- Try coping local file to S3 manually using command
`aws s3 cp /tmp/db.sql s3://mybucket/db.sql`

## Automate the backup
- Create a script for backuping up and uploading automatically
```
#/bin/bash

DATE=$(date +%H-%M-%S)
BACKUP=db-$DATE
DB_HOST=$1
DB_PASSWORD=$2
DB_NAME=$3
AWS_SECRET=%4
BUCKER_NAME=$5
mysqldump -u root -h $DB_HOST -p$DB_PASSWORD $DB_NAME >/tmp/$BACKUP.sql && \
export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE && \
export AWS_SECRET_ACCESS_KEY=AWS_SECRET && \
export AWS_DEFAULT_REGION=us-west-2
echo "Uploading %BACKUP backup"
aws s3 cp /tmp/db-$DATE.sql s3://mybucket/$BACKUP.sql
```

## Managing sensitive info in Jenkins

- Go to Credentials > Jenkins > Global credentials >Add Secret text
- add ID and password for all the secret variables 

## Create Jenkins Job to upload DB to AWS
- Create freeStyle > and add string parameters
- add db_host,aws bucket name and db_name here.
- Check secret text in Build environment
- In Bindings add secret text and bind it to the script variables
- Then execute script with remote host 
- run the script passing the variables in correct order

## Note
- To remove a service use command
 >`docker rm -fv service_name`
- To mount the files permanently use volumes in docker-compose 
```
version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
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
      context: centos7
    volumes:
      - "$PWD/aws-s3.sh:/tmp/script.sh"
    networks:
      - net
  db_host:
    container_name: db
    image: mysql:5.7
    environment:
      - "MYSQL_ROOT_PASSWORD=2310"
    volumes:
      - "$PWD/db_data:/var/lib/mysql"
    networks:
      - net
networks:
  net:
```
- Also add executable permissions to script
> `chmod +x script.sh`
- You can also reuse the same Job to different DB and bucket
