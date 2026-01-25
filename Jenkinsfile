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
        // Requiere: SonarQube configurado en Jenkins (Nombre de instalación) + token en credentials
        // Ajusta estos 2 nombres a lo que tengas en Jenkins:
        withSonarQubeEnv('SonarQube') {
          withCredentials([string(credentialsId: 'jenkins', variable: 'squ_8909dd2b9d68aada09d4ae14176fdc9799dc2caf')]) {
            sh '''
              mvn -B sonar:sonar \
                -Dsonar.login=$SONAR_TOKEN \
                -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
            '''
          }
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
