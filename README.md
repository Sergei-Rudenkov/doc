# Documentation for installiation of ecosustem for integration tests (git server, jenkins, docker)


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
2.2 Install all suggested be default plugins via Jenkins UI localhost:8080
2.3 Add Jenkins ssh key to git user. 
```sh
   sudo cat /var/lib/jenkins/.ssh/id_rsa.pub >> /home/git/.ssh/authorized_keys
```
2.4 Configure version control plugin:
!!!!!!!!!!!!1 picture
2.5 Configure maven plugin
!!!!!!!!!!!!!!!! picture

#### 3. Install Docker 
