Create a JENKINS Pipeline to automate project build process using below Tools

1.Jenkins
2.GitHub
3.Maven
4.Sonarqube
5.Nexus
6.Docker

steps:

1.Create Jenkins-server in aws/gcp

connect to mobaxterm using public ip

install java and jenkins

sudo systemctl enable jenkins

sudo systemctl start jenkins

sudo systemctl status jenkins

http://public-ip:8080/

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Create Admin Account & Install Required Plugins in Jenkins

2.create nexus-server in vm

connect to mobaxtrm using public ip

Install Docker In Ubuntu VM-----

Run Nexus using docker image:

docker run -d -p 8081:8081 --name nexus sonatype/nexus3

 http://public-ip:8081/

Get nexus passsword from Docker Container

sudo docker ps

 docker exec -it <container-id> /bin/bash

 cat /nexus-data/admin.password 

Login into Nexus Server & Reset pwd

3.create sonar-server in aws/gcp

connect using mobaxtrm using public ip

Install Docker In Ubuntu VM-----

docker -v

Run SonarQube using docker image

docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube:lts-community

Enable 9000 port number in Security Group Inbound Rules & Access Sonar Server

bydefault user-admin

         passwd-admin----->login and update password

pipeline stages
------------
node {
}
----------
1.Goto jenkin dashboard -->click on new item-->enter name-demo_pipeline-->select item type-pipeline-->go to pipeline section-->stage('clone reop') {}--------->using pipeline syntax-->select--git:Git--enter git repo url--branch--your branch name--(main)--add--Jenkins--add credential--kind as username and password--ID-for example(GIt-Credential)--generate pipeline syntax--copy and paste inside stage--apply and save--Build Now--see stage view
2.stage('Maven build') { }--goto Mange Jenkins--tools--add maven--select version and give some name(maven-3.9.11)--apply and save--come to pipeline section--
  stage("maven build") {
        def mavenHome = tool name: "maven-3.9.11",type: "maven"
        def mavenCMD = "${mavenHome}/bin/mvn"
        sh "${mavenCMD} clean package"
    }

3.stage('code review') {}---Start SonarQube Server

Login → Generate Sonar Token (Example: sqa_7295ab51385a8042093f97920b44229e6d83609c)

Add Token in Jenkins Credentials → Secret Text

Install SonarQube Scanner Plugin in Jenkins

Configure SonarQube Server in system:

Name: sonar-server-7.8

URL: http://<Sonar-IP>:9000/

Token: 

Add Sonar Stage
 stage('code review') {
        withSonarQubeEnv('sonar-server-7.8') {
        def mavenHome = tool name: "maven-3.9.11",type: "maven"
        def mavenCMD = "${mavenHome}/bin/mvn"
        sh "${mavenCMD} sonar:sonar"
        }
    }
goto sonar server in browser and see project code review

4.Upload Artifact to Nexus

Create Maven (hosted) repository in Nexus

Install Nexus Artifact Uploader Plugin in Jenkins

Generate Pipeline Syntax → Nexus Artifact Upload
Nexus Version:
Protocol:HTTP
Nexus URL:34.69.17.203:8081/
Credentials
set up jenkin credential using kind as username and password--ID--nexus-credential--
GroupId:same as in pom.xml
Version:3.0-SNAPSHOT (same as in pom.xml)
Repository:pk-snapshot-repo
add Artifact
ArtifactId:01-maven-web-app
Type:war
classifier:---
file:target/01-maven-web-app.war
Generate pipeline syntax:
nexusArtifactUploader artifacts: [[artifactId: '01-maven-web-app', classifier: '', file: 'target/01-maven-web-app.war', type: 'war']], credentialsId: 'nexus-credential', groupId: 'in.ashokit', nexusUrl: '34.69.17.203:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'pk-snapshot-repo', version: '3.0-SNAPSHOT'

save and apply--goto nexus repo and see uploaded artifact
5.Create Docker Image
Add the Jenkins user to the docker group:
sudo usermod -aG docker jenkins
Then restart Jenkins:
sudo systemctl restart jenkins
*After this step run pipeline without sudo comand
  stage('docker image') {
        sh 'docker build -t vermakqr/mavenwebapp .'
    }

go to jenkins vm and see docker image created
$sudo docker images
  stage('docker image') {
        sh 'docker build -t vermakqr/mavenwebapp .'
    }

6. Upload Image to Registry

 usinig pipeline syntax ---secret key and store in variable

     stage("upload image") {
         withCredentials([string(credentialsId: 'docker-credential', variable: 'dockerHubPwd')]) {
          sh '''
          docker login -u vermakqr -p "${dockerHubPwd}"
          docker build -t vermakqr/mavenwebapp .
          '''
         }
        }
finally docker image uploaded to registry

7.Deploy stage

 stage('deploy') {
        sh 'docker run -d -p 8082:8080 vermakqr/mavenwebapp'
    }

Enable 8082 port in Security group

http://public-ip of docker-vm:8082/context-path ---->

------------------------------------------------------------------------------------------------------------
final script
------------------------------------------------------------------------------------------------------------

node {
    stage('git clone') {
        git branch: 'main', credentialsId: 'Git-Credentials', url: 'https://github.com/dev-verma/maven-web-app.git'

    }
    stage("maven build") {
        def mavenHome = tool name: "maven-3.9.11",type: "maven"
        def mavenCMD = "${mavenHome}/bin/mvn"
        sh "${mavenCMD} clean package"
    }
    stage('code review') {
        withSonarQubeEnv('sonar-server-7.8') {
        def mavenHome = tool name: "maven-3.9.11",type: "maven"
        def mavenCMD = "${mavenHome}/bin/mvn"
        sh "${mavenCMD} sonar:sonar"
        }
    }
    stage('Artifact upload') {
        nexusArtifactUploader artifacts: [[artifactId: '01-maven-web-app', classifier: '', file: 'target/01-maven-web-app.war', type: 'war']], credentialsId: 'nexus-credential', groupId: 'in.ashokit', nexusUrl: '34.69.17.203:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'pk-snapshot-repo', version: '3.0-SNAPSHOT'
    }
    stage('docker image') {
        sh 'docker build -t vermakqr/mavenwebapp .'
    }
    stage("upload image") {
         withCredentials([string(credentialsId: 'docker-credential', variable: 'dockerHubPwd')]) {
          sh '''
          docker login -u vermakqr -p "${dockerHubPwd}"
          docker build -t vermakqr/mavenwebapp .
          '''
         }
    }
    stage('deploy') {
        sh 'docker run -d -p 8082:8080 vermakqr/mavenwebapp'
    }
} 

