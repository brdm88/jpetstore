pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo 'Starting Build Step'
        sh 'mvn clean install -Dlicense.skip=true'
        echo 'Build Step Completed'
      }
    }

    stage('Testing') {
      parallel {
        stage('Sonarqube Test') {
          steps {
            sh 'mvn sonar:sonar -Dsonar.host.url=http://sonarqube:8081 -Dlicense.skip=true'
          }
        }

        stage('Print Tester Credentials') {
          steps {
            echo "The tester is ${TESTER}"
          }
        }

        stage('Print Build Number') {
          steps {
            echo "This is build number ${BUILD_ID}"
            sleep 20
          }
        }

      }
    }

    stage('JFrog Push') {
      steps {
        script {
          def server = Artifactory.server "artifactory"


          def buildInfo = Artifactory.newBuildInfo()
          def rtMaven = Artifactory.newMavenBuild()

          rtMaven.tool = 'maven'
          rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'

          buildInfo = rtMaven.run pom: 'pom.xml', goals: "clean install -Dlicense.skip=true"
          buildInfo.env.capture = true
          buildInfo.name = 'jpetstore-6'
          server.publishBuildInfo buildInfo
        }

      }
    }

    stage('Deploy Prompt') {
      steps {
        input 'Deploy to Production?'
      }
    }

    stage('Deployment') {
      steps {
        echo 'Deploy Stage'
      }
    }

  }
  tools {
    maven 'maven'
  }
  environment {
    TESTER = 'TheBRDM'
  }
}