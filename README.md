## Continuous-Integration-Using-Jenkins-Nexus-Sonarqube-Slack

We will build Continuous Integration pipeline and following tools will be used

* Jenkins > Continuous Integration Server
* GitHub > Remote Repository
* Maven > Build Tool
* Checkstyle > Code Analysis Tool
* Slack > Notification Tool
* SonaType Nexus > Artifact Repository
* SonarQube > Code Analysis Server 
* AWS EC2 > Compute Resource

### Workflow

[![Screenshot-2022-10-23-at-23-19-11.png](https://i.postimg.cc/nzJ15MXK/Screenshot-2022-10-23-at-23-19-11.png)](https://postimg.cc/xk6HNfyq)

### Onboarding Servers

We will launch 3 EC2 instances which will host Jenkins Server,SonarQube Server and Nexus Server. We will use script via user-data in order to have smooth and fast onboarding of servers. Scripts are located in userdata directory on repository. We will using Ubuntu for Jenkins and SonarQube ,for Nexus will use Amazon Linux 2. Let's do small verification

For Jenkins

```
ubuntu@ip-172-18-170-134:~$ systemctl status jenkins
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-10-24 19:32:29 UTC; 3min 34s ago
   Main PID: 5635 (java)
      Tasks: 36 (limit: 2351)
     Memory: 360.7M
     CGroup: /system.slice/jenkins.service
             └─5635 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webro>

```

We will install following plugins on Jenkins

* Maven Integration
* GitHub Integration
* Nexus Artifact Uploader
* SonarQube Scanner
* Slack Notification
* Build Timestamp (for doing versioning on artifact)

For Nexus

```
[root@ip-172-18-154-251 ~]# systemctl status nexus
● nexus.service - nexus service
   Loaded: loaded (/etc/systemd/system/nexus.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-10-24 19:32:54 UTC; 16min ago
 Main PID: 3906 (java)
   CGroup: /system.slice/nexus.service
           └─3906 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.342.b07-1.amzn2.0.1.x86_64/jre/bin/java -...

Oct 24 19:32:54 ip-172-18-154-251.ec2.internal systemd[1]: Starting nexus service...
Oct 24 19:32:54 ip-172-18-154-251.ec2.internal nexus[3700]: Starting nexus
Oct 24 19:32:54 ip-172-18-154-251.ec2.internal systemd[1]: Started nexus service.
[root@ip-172-18-154-251 ~]# 

```
We will create repositories with following name on Sonatype Nexus


* devops-release >> for storing artifacts
* devops-maven-central >> will be used for downloading dependencies for Maven from Jenkins.
* devops-snapshot >> will be used in case of snapshots
* devops-group >> to group all repositories

### Configuration on Jenkins

####  Global Tool Configuration
We will need to enable following Tools on Jenkins in order to run successfull pipeline

* Oracle JDK installation (we will need to provide our private Oracle credentials in order to download JDK from Oracle)
* Maven installation

#### Credentials

We will need to add Nexus Credentials on Jenkins












