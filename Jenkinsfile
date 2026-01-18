pipeline {
  agent any
  tools {
    jdk 'JDK17'
    maven 'Maven3'
  }

  stages {
    stage('Build & Test') {
      steps {
        sh 'echo JAVA_HOME=$JAVA_HOME'
        sh 'java -version'
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
