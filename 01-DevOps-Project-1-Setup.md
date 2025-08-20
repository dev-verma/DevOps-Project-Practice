🚀 DevOps Project Setup with CI/CD Pipeline

This project demonstrates End-to-End CI/CD pipeline using:

GitHub (Source Code Management)

Maven (Build Tool)

SonarQube (Code Quality)

Nexus Repo (Artifact Repository)

Tomcat (Application Server)

Jenkins (CI/CD Orchestrator)

✅ Step-0: Create Jenkins Pipeline (Scripted Pipeline)
node {

}

✅ Step-1: Clone Source Code from GitHub

Generate Pipeline Syntax → Git SCM → Copy Script

git credentialsId: 'GIT-Credentials', url: 'your-github-url'


Add Git Stage in Pipeline

stage('Clone Repo') {        
    git credentialsId: 'GIT-Credentials', url: 'your-github-url'
}

✅ Step-2: Build with Maven

Configure Maven as Global Tool:

Jenkins Dashboard → Manage Jenkins → Global Tool Configuration → Add Maven

Add Build Stage

stage('Maven Build') {
    def mavenHome = tool name: "Maven-3.9.4", type: "maven"
    def mavenCMD = "${mavenHome}/bin/mvn"
    sh "${mavenCMD} clean package"
}

✅ Step-3: SonarQube Code Quality Analysis

Start SonarQube Server

Login → Generate Sonar Token (Example: cedbc0b89e45c58f4a86e4687f2df2a2241e3369)

Add Token in Jenkins Credentials → Secret Text

Install SonarQube Scanner Plugin in Jenkins

Configure SonarQube Server:

Name: Sonar-Server-7.8

URL: http://<Sonar-IP>:9000/

Token: (From Step-3)

Add Sonar Stage

stage('SonarQube Analysis') {
    withSonarQubeEnv('Sonar-Server-7.8') {
        def mavenHome = tool name: "Maven-3.8.6", type: "maven"
        def mavenCMD = "${mavenHome}/bin/mvn"
        sh "${mavenCMD} sonar:sonar"
    }
}

✅ Step-4: Upload Artifact to Nexus

Run Nexus Server

Create Maven (hosted) repository in Nexus

Install Nexus Artifact Uploader Plugin in Jenkins

Generate Pipeline Syntax → Nexus Artifact Upload

stage('Nexus Upload') {
    nexusArtifactUploader artifacts: [[artifactId: '01-Maven-Web-App', classifier: '', file: 'target/01-maven-web-app.war', type: 'war']], 
    credentialsId: 'Nexus-Credentials', 
    groupId: 'in.xyz', 
    nexusUrl: '13.127.185.241:8081', 
    nexusVersion: 'nexus3', 
    protocol: 'http', 
    repository: 'demo-snapshot-repository', 
    version: '1.0-SNAPSHOT'
}

✅ Step-5: Deploy to Tomcat Server

Start Tomcat Server

Install SSH Agent Plugin in Jenkins

Add Tomcat Server Credentials in Jenkins

Configure Deployment Stage

stage('Deploy'){ 
    sshagent(['Tomcat-Server-Agent']) {
        sh 'scp -o StrictHostKeyChecking=no target/01-maven-web-app.war ec2-user@15.207.100.83:/home/ec2-user/apache-tomcat-9.0.80/webapps'
    }
}
---------------------------------------------------------------------------------------------------------------------------------------------------------------
📌 Final Jenkins Pipeline (Full Script)
--------------------------------------------
node {

    stage('Clone Repo') {        
        git credentialsId: 'GIT-Credentials', url: 'github-url'
    }

    stage('Maven Build') {
        def mavenHome = tool name: "Maven-3.9.4", type: "maven"
        def mavenCMD = "${mavenHome}/bin/mvn"
        sh "${mavenCMD} clean package"
    }

    stage('SonarQube Analysis') {
        withSonarQubeEnv('Sonar-Server-7.8') {
            def mavenHome = tool name: "Maven-3.8.6", type: "maven"
            def mavenCMD = "${mavenHome}/bin/mvn"
            sh "${mavenCMD} sonar:sonar"
        }
    }

    stage('Nexus Upload') {
        nexusArtifactUploader artifacts: [[artifactId: '01-Maven-Web-App', classifier: '', file: 'target/01-maven-web-app.war', type: 'war']], 
        credentialsId: 'Nexus-Credentials', 
        groupId: 'in.ashokit', 
        nexusUrl: '13.127.185.241:8081', 
        nexusVersion: 'nexus3', 
        protocol: 'http', 
        repository: 'ashokit-snapshot-repository', 
        version: '1.0-SNAPSHOT'
    }

    stage('Deploy'){ 
        sshagent(['Tomcat-Server-Agent']) {
            sh 'scp -o StrictHostKeyChecking=no target/01-maven-web-app.war ec2-user@15.207.100.83:/home/ec2-user/apache-tomcat-9.0.80/webapps'
        }
    }
}


