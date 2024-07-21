pipeline {
  agent {
    node {
      label 'agentNode1'
    }
  }
  
  stages {
    stage('Build') {
      steps {
        sh './mvnw clean compile'
      }
    }

    stage('Static Code Analysis') {
      steps {
        sh '''./mvnw sonar:sonar \\
  -Dsonar.projectKey=SprintPet \\
  -Dsonar.projectName=\'SprintPet\' \\
  -Dsonar.host.url=http://ec2-3-110-229-202.ap-south-1.compute.amazonaws.com:9000 \\
  -Dsonar.token=sqp_1758f055ba470489227b47ac676f4dc011e25492'''
      }
    }

    stage('Unit Tests') {
      steps {
        sh './mvnw "-Dtest=**/petclinic/*/*.java" test'
        junit '**/target/surefire-reports/TEST-*.xml'
      }
    }

    stage('Create Package') {
      steps {
        sh './mvnw package -DskipTests=true'
      }
    }

    stage('Upload JAR to Nexus') {
      steps {
        script {
          nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: 'http://ec2-3-110-229-202.ap-south-1.compute.amazonaws.com:8081',
            repository: 'SpringPet',
            groupId: 'Dev',
            version: "{env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
            credentialsId: '1e30c233-ab1e-4d55-b70b-018c5b977ed3',
            artifacts: [
              [ artifactId: 'SpringPet',
                classifier: '',
                file: "${WORKSPACE}/target/spring-petclinic-2.7.3.jar",
                type: 'jar' ]
            ]
          )
        }
      }
    }

    stage('Parallel Deploy and Test') {
      parallel {
        stage('Deploy') {
          steps {
            sh './mvnw spring-boot:run </dev/null &>/dev/null &'
          }
        }

        stage('Integration and Performance Tests') {
          steps {
            sh './mvnw verify'
            junit '**/target/surefire-reports/TEST-*.xml'
            perfReport(sourceDataFiles: '**/target/jmeter/results/*', showTrendGraphs: true, ignoreFailedBuilds: true, ignoreUnstableBuilds: true)
          }
        }

      }
    }
  }
}
