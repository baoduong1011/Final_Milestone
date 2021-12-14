pipeline {
  agent any

  tools {
    jdk 'JDK11'
    maven 'Milestone_Maven' 
  }

  stages {
    stage('Build And Test Stage') {
      steps {
        withMaven(maven : 'Milestone_Maven') {
          sh "mvn package"
        }
      }
    }
  
//      stage ('OWASP Check Vulnerabilities Stage') {
//       steps {
//         dependencyCheck additionalArguments: ''' 
//           -o "./" 
//           -s "./"
//           -f "ALL" 
//           --prettyPrint''', odcInstallation: 'Dependency_Milestone'

//         dependencyCheckPublisher pattern: 'dependency-check-report.xml'
//       }
//     }
    stage('SonarQube Analysis Stage') {
      environment {
                scannerHome = tool 'Milestone_Sonar'
      }
      steps {
        withSonarQubeEnv('Milestone_SonarServer') {
          withMaven(maven : 'Milestone_Maven') {
            sh 'mvn sonar:sonar -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html'
          }
        }
      }
    }
    stage("Quality Gate Stage") {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    stage('Create and push container Stage') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
          withMaven(maven : 'Milestone_Maven') {
            sh "mvn jib:build"
          }
        }
      } 
    }

//     stage('Anchore analyse') {
//       steps {
//         writeFile file: 'anchore_images', text: 'docker.io/maartensmeets/spring-boot-demo'
//         anchore name: 'anchore_images'
//       }
//     }


  }
}
