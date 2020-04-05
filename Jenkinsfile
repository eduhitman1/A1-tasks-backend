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
    }

}


