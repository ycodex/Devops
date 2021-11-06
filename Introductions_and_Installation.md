# Setup the lab

- Download Oracle Virtual box and CentOs 7 (Preferred minimal version)
- Install the Centos ISO in Virtual box
- Download the putty and configure by giving it ip address of centos 
	> type `ip a` command and get the inet address
- Install Docker and Docker-compose in the system and
- using the create a docker-compose.yml file for creating Jenkins container
- The create the jenkins container using docker-compose , check the template for reference.
	> use  `docker-compose up -d` command to get it running
	> check with  `docker ps` command 
- Check the container is running in browser using the ip address with the post number
- Create a local DNS to access the jenkins site easily
	> For this goto `C:\Windows\System32\drivers\etc` and add the ip and host name


## Resources

```version: '3' 
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
networks:
  net:
```

