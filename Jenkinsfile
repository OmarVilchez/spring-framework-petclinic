pipeline {
  agent {
    docker {
      image 'maven:3.9-eclipse-temurin-17'
      args '-v $HOME/.m2:/root/.m2'
    }
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build & Test') {
      steps {
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
