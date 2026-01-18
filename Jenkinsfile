pipeline {
  agent any
  tools {
    maven 'Maven3'
    jdk 'JDK17'
  }
  stages {
    stage('Build & Test') {
      steps {
        sh 'mvn -v'
        sh 'mvn -B clean test'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        }
      }
    }
  }
}
