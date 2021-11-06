# Jenkins And Docker

## Docker+Jenkins+SSH
- Create another container for which Jenkins will be conneting it 
- Create a ssh key for connecting without password
	>For creating it use `ssh-keygen -f remote-key
	> i am storing it all in a folder called centos7
- So for creating container create a Dockerfile like below
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
CMD /usr/sbin/sshd -D 
```
- Next create a container using the Dockerfile in docker compose. The docker compose file will be something like below
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
networks:
  net:

```
- Then spin up the containers
- In Jenkins UI install SSH plugin
- Go to manage jenkins>configure system>ssh remote host
- Then add host name and the the private key that was generated with adding credentials
- Check for successfull connection.
- Create and run the job with build as execute script on remote host
- Finally Check for the execution in remote host.
