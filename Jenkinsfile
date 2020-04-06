pipeline{
    agent any
    stages{
        stage('Build Backend'){
           steps{
               bat 'mvn clean package -DskipTests=true'
           }
        }

        stage('Unit Test'){
           steps{
               bat 'mvn test'
           }
        }

        stage('Sonar Analysis'){
           environment{
               scannerHome = tool 'SONAR_SCANNER'
           }
           steps{
               withSonarQubeEnv('SONAR_LOCAL'){
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000  -Dsonar.login=9fd1825804f055763ea5818cc61b5837e1b5d6bd -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
               }
           }
        }

        stage('Quality Gate'){
            steps{
                sleep(5)
                timeout(time:1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy Backend'){
            steps{
                 deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'                
                }
        }
        
        stage('API Test'){
            steps{
                dir('api-test') {
                     git credentialsId: 'login_github', url: 'https://github.com/eduhitman1/A1-tasks-api-test'
                     bat 'mvn test'
                    }
                }
       }

       stage('Deploy Frontend'){
            steps{
                 dir('frontend'){
                 git credentialsId: 'login_github', url: 'https://github.com/eduhitman1/A1-tasks-frontend'
                 bat 'mvn clean package'
                 deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'        
                 }
                }
       }

        
       stage('Funcional Test'){
            steps{
                dir('funcional-test') {
                     git credentialsId: 'login_github', url: 'https://github.com/eduhitman1/A1-funcional-tasks'
                     bat 'mvn test'
                    }
                }
        }

        stage('Deploy Prod'){
            steps{
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }

        stage('Health Check'){
            steps{
                sleep(5)
                dir('funcional-test') {
                     bat 'mvn test -Dskip.surefire.tests'
                    }
                }
        }

    }

    post{
        always{
             junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, funcional-test/target/surefire-reports/*.xml, funcional-test/target/failsafe-reports/*.xml'  
             archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war ', onlyIfSuccessful: true
        }

       unsuccessful{
             emailext attachLog: true, body: 'See the attached log bellow', subject: 'Build $BUILD NUMBER has failed', to: 'edu.hitman01.eh+jenkins@gmail.com'
       }

        fixed{
             junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, funcional-test/target/surefire-reports/*.xml, funcional-test/target/failsafe-reports/*.xml'  
             emailext attachLog: true, body: 'See the attached log bellow', subject: 'Build is fine!!', to: 'edu.hitman01.eh+jenkins@gmail.com'
        }
    }

}