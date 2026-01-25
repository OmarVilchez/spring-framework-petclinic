pipeline {
  agent any

  tools {
    jdk 'JDK17'
    maven 'Maven3'
  }

  stages {

    stage('Build') {
      steps {
        sh 'echo JAVA_HOME=$JAVA_HOME'
        sh 'java -version'
        sh 'mvn -v'
        // Build sin ejecutar tests (rápido). Si prefieres compilar + empaquetar, cambia a package.
        sh 'mvn -B -U -DskipTests clean package'
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: true
        }
      }
    }

    stage('Testing (JUnit + JaCoCo)') {
      steps {
        // Ejecuta tests + genera reporte JaCoCo (XML recomendado para Sonar)
        // Si tu POM ya liga jacoco a la fase test/verify, esto igual funciona y solo fuerza el report.
        sh 'mvn -B test jacoco:report'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'

          // Requiere plugin "JaCoCo" en Jenkins
          jacoco(
            execPattern: '**/target/*.exec,**/target/jacoco.exec',
            classPattern: '**/target/classes',
            sourcePattern: '**/src/main/java',
            exclusionPattern: '**/target/test-classes/**'
          )
        }
      }
    }

    stage('Sonar') {
      steps {
        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
          sh """
            mvn -B -ntp sonar:sonar \
              -Dsonar.host.url=http://138.68.0.100:9000 \
              -Dsonar.login=${SONAR_TOKEN} \
              -Dsonar.projectKey=petclinic-monolith \
              -Dsonar.projectName=Spring PetClinic Monolith \
              -Dsonar.projectVersion=1.0 \
              -Dsonar.sources=src/main \
              -Dsonar.tests=src/test \
              -Dsonar.java.binaries=target/classes
          """
        }
      }
    }


    // Opcional recomendado: rompe el pipeline si falla el Quality Gate
    stage('Quality Gate') {
      steps {
        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }
}
