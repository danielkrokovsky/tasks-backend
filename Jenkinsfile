pipeline {
    agent any
    stages{
        stage('Build Backend'){
            steps{
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Sonar Analisys'){
            environment{
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps{
                withSonarQubeEnv('SONAR_LOCAL'){
                    sh '${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=3e61847f199893bb6cb0f3cb5b3ee314715e6f1e -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java'
                }
            }
        }
        stage('Quality Gate'){
            steps{
                sleep(5)
                timeout(time: 1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true, credentialsId: 'SONAR_TOKEN'
                }
            }
        }
        stage('Deploy BackEnd'){
            steps{
                deploy adapters: [tomcat8(credentialsId: 'f86a9675-12cf-4f09-acbe-03db3086c4e3', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }

        stage('Deploy FrontEnd'){
            steps{
                dir('frontend'){
                    git 'https://github.com/danielkrokovsky/tasks-frontend.git'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'f86a9675-12cf-4f09-acbe-03db3086c4e3', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
    }
        post{
            always{
                junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
            }
            unsuccessful{
                emailext body: 'Build falhou', subject: 'Build has failed', to: 'danielkrokovsky@hotmail.com'
            }
            fixed{
                emailext body: '', subject: 'Build is fine', to: 'danielkrokovsky@hotmail.com'
            }
    }
}