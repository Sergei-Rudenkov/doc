# Documentation for installiation of ecosystem for integration tests (git server, jenkins, docker)


Primary invironment:
  - Ubuntu 15.10

#### 1. Git server instoliation 
You don't need local git server if you have a remote one. See the [video tutorial](https://www.youtube.com/watch?v=lXSZUuDW4nY) on local git server installiation.
1.1. Install git client
```sh 
  sudo apt-get install git
```  
1.2. Create git user:
```sh
   sudo adduser git
```
1.3. Create directory for remote source, init git 
```sh
   sudo mkdir -R /opt/git/my_project.git
   cd /opt/git/my_project.git
   cd my_project.git
   git --bare init
   sudo chown -R git:git /opt/git # changed ownership 
```
1.4. Add ssh
```sh
   su git    
   cd
   mkdir .ssh
```
1.5. Add ssh keys 
```
   cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
```
See documentation [here](https://git-scm.com/book/ru/v1/Git-%D0%BD%D0%B0-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%B5-%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%B0%D0%B8%D0%B2%D0%B0%D0%B5%D0%BC-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80)

#### 2. Jenkins installiation
2.1 Install Jenkins on Ubuntu
``` sh
   wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   sudo apt-get update
   sudo apt-get install jenkins
```
2.2 Install all plugins suggested being default via Jenkins UI (usually localhost:8080)
2.3 Add Jenkins ssh key to git user. 
```sh
   sudo cat /var/lib/jenkins/.ssh/id_rsa.pub >> /home/git/.ssh/authorized_keys
```
2.4 Configure version control plugin:

![vc congig](https://raw.githubusercontent.com/Sergei-Rudenkov/temp_documentation/master/Selection_089.png)

2.5 Configure maven plugin

![Maven plugin config](https://raw.githubusercontent.com/Sergei-Rudenkov/temp_documentation/master/Selection_091.png)

#### 3. Configure Docker 

3.1 Please install Docker according to this [instructions](https://docs.docker.com/engine/installation/linux/ubuntulinux/) 

3.2 Create `Dockerfile` with configuration  

```sh
FROM jboss/base-jdk:7
ADD jboss-eap-6.1.1.zip /tmp/
RUN unzip /tmp/jboss-eap-6.1.1.zip -d /opt/jboss

# Add EAP_HOME environment variable, to easily upgrade the script for different EAP versions
ENV EAP_HOME /opt/jboss/jboss-eap-6.1

# Add customized data and module librares put it to the root of Dockerfile 
ADD application-roles.properties /opt/jboss/jboss-eap-6.1/standalone/configuration/
ADD application-users.properties /opt/jboss/jboss-eap-6.1/standalone/configuration/
ADD project.ear /opt/jboss/jboss-eap-6.1/standalone/deployments/
ADD standalone.xml /opt/jboss/jboss-eap-6.1/standalone/configuration/
ADD logging.properties /opt/jboss/jboss-eap-6.1/standalone/configuration/
ADD modules /opt/jboss/jboss-eap-6.1/modules/

# Add default admin user
RUN ./opt/jboss/jboss-eap-6.1.1/bin/add-user.sh admin admin123! --silent

# Enable binding to all network interfaces and debugging inside the EAP
RUN echo "JAVA_OPTS=\"\$JAVA_OPTS -Dcom.sun.jersey.server.impl.cdi.lookupExtensionInBeanManager=true -Djboss.server.base.dir=/opt/jboss/jboss-eap-6.1/standalone -Djboss.server.name=standalone\"" >> ${EAP_HOME}/bin/standalone.conf
# Add volume if you want to externalize logs
VOLUME ${EAP_HOME}/standalone/logs
EXPOSE 55280

ENTRYPOINT ["/opt/jboss/jboss-eap-6.1/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]
CMD []
```

3.3. Create build.sh that will build Docker image, run it.

```sh
#!/bin/bash

docker build -t my-docker-image/jboss-eap:6.1.1  .
```

3.4. Run container from image. 
```sh
docker run -p 56280:56280 --add-host=docker:10.10.10.10 -it --rm my-docker-image/jboss-eap:6.1.1
```
* --add-host=docker:10.10.10.10 - external ip to DB server 

* -p 56280:56280 - exposing port to host machine 

__Note!__ To run docker commanly you need sudo rights, or [add user to Docker Group](http://askubuntu.com/questions/477551/how-can-i-use-docker-without-sudo). Since Jenkins is not a user, but [service account](http://stackoverflow.com/a/18081006/3014866) you can not do any of this. You can overcome it by adding any user to docker group and run from Jenkins using ssh connection to user, I run it from git user hence we already added Jenkins ssh key to git user in the step _2.3_.

```sh
ssh -tt git@myworkplace docker run -p 56280:56280 --add-host=docker:10.6.210.32 -it --rm my-docker-image/jboss-eap:6.1.1
```
