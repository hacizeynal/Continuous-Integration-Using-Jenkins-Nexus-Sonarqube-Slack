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
        NEXUS_LOGIN= "nexuslogin"

    }

    stages{
        stage("Build"){
            //step to pass Unit Tests and pass parameters from setting.xml and install dependencies.
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
        }
    }
}