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
        withSonarQubeEnv('sonar') {
          sh """
            mvn -B -ntp verify \
              org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
              -Dsonar.projectKey=petclinic-monolith \
              -Dsonar.projectName='Spring PetClinic Monolith' \
              -Dsonar.projectVersion=1.0
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

    stage('Publish Artifacts') {
      steps {
        script {
          def server = Artifactory.server('artifactory')
          def buildInfo = Artifactory.newBuildInfo()

          def uploadSpec = """{
            "files": [
              {
                "pattern": "target/*.jar",
                "target": "spring-petclinic-rest-release/petclinic/${BUILD_NUMBER}/"
              }
            ]
          }"""

          server.upload(uploadSpec, buildInfo)
          server.publishBuildInfo(buildInfo)
        }
      }
    }

//     stage("Publish Artifacts (Artifactory - File Spec)") {
//       steps {
//         script {
//           def server = Artifactory.server('artifactory')
//           def targetRepo = 'spring-petclinic-rest-release'   // tu repo Maven local
//
//           def pom = readMavenPom file: 'pom.xml'
//           def groupIdPath = pom.groupId.replace('.', '/')
//
//           // PetClinic es WAR normalmente. Por eso sube *.war y también *.jar por si acaso.
//           def uploadSpec = """
//           {
//             "files": [
//               {
//                 "pattern": "target/${pom.artifactId}-${pom.version}.war",
//                 "target": "${targetRepo}/${groupIdPath}/${pom.artifactId}/${pom.version}/",
//                 "flat": "true"
//               },
//               {
//                 "pattern": "target/${pom.artifactId}-${pom.version}.jar",
//                 "target": "${targetRepo}/${groupIdPath}/${pom.artifactId}/${pom.version}/",
//                 "flat": "true"
//               }
//             ]
//           }
//           """
//
//           server.upload spec: uploadSpec
//         }
//       }
//     }






  }
}
