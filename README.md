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

#### GitHub Webhook

In order to run job automatically we should enable GitHub Webhook ,in each commit job will run automatically without human interaction.

GitHub will send a POST request to the URL **(http://ip_address_of_Jenkins:/8080/github-webhook)** with details of any subscribed events. We can also specify which data format you’d like to receive (JSON, x-www-form-urlencoded, etc). 

Please make sure that Jenkins security group is allowing port 8080 from anywhere.

#### Checking Reports

After running successfull pipeline ,in 2nd and 3rd stage will generate report ,we can check those reports on **Workspace/target** directory.

[![Screenshot-2022-10-25-at-23-01-35.png](https://i.postimg.cc/x87zNkKf/Screenshot-2022-10-25-at-23-01-35.png)](https://postimg.cc/y3mdw8Qt)

Those reports are not human readable ,in order to analyse ,store and present them on dashboard we will need tool named SonarQube server.This dashboard will present code analysis and unit tests results.

### Configuration on SonarQube Server

We will need code on Jenkins pipeline which generate report based on code analysis and upload it to the SonarQube Server.We will need following tool and information on Jenkins.

* SonarScanner Tool
* SonarQube Server information

From Jenkins Tool configuration we will need to add SonarQube Scanner ,then we will need to navigate to Configure Jenkins section and add our private ip and token of SonarQube Scanner for integration.

After successfull run of pipeline we can see following logs on console output ,which means that ANALYSIS is successfull and we can check output on our SonarQube Server.

```
INFO: CPD Executor Calculating CPD for 14 files
INFO: CPD Executor CPD calculation finished (done) | time=51ms
INFO: Analysis report generated in 160ms, dir size=262 KB
INFO: Analysis report compressed in 173ms, zip size=98 KB
INFO: Analysis report uploaded in 310ms
INFO: ANALYSIS SUCCESSFUL, you can browse http://172.18.150.252/dashboard?id=vprofile
INFO: Note that you will be able to access the updated dashboard once the server has processed the submitted analysis report
INFO: More about the report processing at http://172.18.150.252/api/ce/task?id=AYQRSZKnkW0txDCYt5pJ
INFO: Analysis total time: 16.003 s
INFO: ---------------------------------------------------------

```
[![Screenshot-2022-10-26-at-00-39-04.png](https://i.postimg.cc/3rgCbhgd/Screenshot-2022-10-26-at-00-39-04.png)](https://postimg.cc/YjCWhJDH)


#### Setting up Quality Sonar Gate

We can configure Quality Sonar Gates on SonarQube and set up some rules and conditions ,at the end of pipeline if our rule will not pass our pipeline job will fail. For example ,we can set a condition that our pipeline will fail if we will have more than 25 bugs. I have added new condition that if Bug count is greater than 25 Job will fail. Let's verify.

```
Timeout set to expire in 1 hr 0 min
[Pipeline] {
[Pipeline] waitForQualityGate
Checking status of SonarQube task 'AYQTIptAkW0txDCYt5pU' on server 'sonarqubescanner'
SonarQube task 'AYQTIptAkW0txDCYt5pU' status is 'PENDING'
SonarQube task 'AYQTIptAkW0txDCYt5pU' status is 'SUCCESS'
SonarQube task 'AYQTIptAkW0txDCYt5pU' completed. Quality gate is 'ERROR'
[Pipeline] }
[Pipeline] // timeout
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
ERROR: Pipeline aborted due to quality gate failure: ERROR
Finished: FAILURE

```

[![Screenshot-2022-10-26-at-09-27-07.png](https://i.postimg.cc/JtV4KxsF/Screenshot-2022-10-26-at-09-27-07.png)](https://postimg.cc/wR2pjhc5)

#### Setting up Build Timestamp

We will enable Build Timestamp Plugin in order to add versioning tag to the different version of Artifacts and we will upload all those artifacts to the Nexus Sonartype Repository.
After successful running of pipeline we can confirm that artifacts **BUILD_ID 20 and 21** are uploaded to Nexus Artifact Repository.

[![Screenshot-2022-10-26-at-12-26-23.png](https://i.postimg.cc/PxG16L7C/Screenshot-2022-10-26-at-12-26-23.png)](https://postimg.cc/McDcHGqw)

[url=https://postimg.cc/686xw6jn][img]https://i.postimg.cc/686xw6jn/Screenshot-2022-10-26-at-12-27-17.png[/img][/url]







