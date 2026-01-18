pipeline {
  agent any
  stages {
    stage('Build & Test') {
      steps {
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
