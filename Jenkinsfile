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
      credentialsId = '1e30c233-ab1e-4d55-b70b-018c5b977ed3' // Replace with your credential ID
      steps {
        script {
          def nexusUrl = 'http://ec2-3-110-229-202.ap-south-1.compute.amazonaws.com:8081' // Replace with your Nexus URL
          def repo = 'SpringPet' // Replace with your repository name
          nexusUploader(
            nexusUrl: nexusUrl,
            repository: repo,
            credentialsId: credentialsId,
            artifacts: [[ file: "${WORKSPACE}/target/spring-petclinic*.jar" ]] // Upload all JARs in target folder
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
