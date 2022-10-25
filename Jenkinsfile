pipeline{
    agent any
    tools{
        maven "MAVEN3"
        jdk "ORACLEJDK8"
    }
    environment{
        SNAP_REPO = "devops-snapshot"
        NEXUS_USER = "admin"
        NEXUS_PASS = "Krakow123"
        RELEASE_REPO = "devops-release"
        CENTRAL_REPO = "devops-maven-central"
        NEXUSIP = "172.18.154.251"
        NEXUSPORT = "8081"
        NEXUS_GRP_REPO = "devops-group"
        NEXUS_LOGIN = "nexuslogin"
        SONARSCANNER = "sonarqubescanner"
        SONARSERVER = "sonarserver"

    }

    stages{
        stage("BUILD"){
            //step to skip Unit Tests and pass parameters from settings.xml and install dependencies locally.
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post{
                success {
                echo "Now archiving"
                archiveArtifacts artifacts: '**/*.war'

                }
            }
        }
        stage("UNIT TEST"){
            // Execute Unit Test
            steps {
                sh 'mvn -s settings.xml test'
            }


        }
        stage("CODE ANALYSIS WITH CHECKSTYLE"){
            // Execute Checkstyle analysis and generates a report on violations.
            steps {
                sh "mvn -s settings.xml checkstyle:checkstyle"
            }    
    
        }
        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool "${SONARSCANNER}"
          }

          steps {
            withSonarQubeEnv("${SONARSCANNER}") {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }
    }
}