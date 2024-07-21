pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh './mvnw clean compile'
      }
    }
    stage('Static Code Analyzer') {
      steps {
        sh './mvnw sonar:sonar -Dsonar.projectKey=SprintPet -Dsonar.projectName=SprintPet -Dsonar.host.url=http://ec2-3-110-229-202.ap-south-1.compute.amazonaws.com:9000 -Dsonar.token=sqp_1758f055ba470489227b47ac676f4dc011e25492'
      }
    }

  }
}
