pipeline {
    agent any
    stages {
        stage ('Construção do backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Testes unitários') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Análise estática via SonarQube') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=833910a98e879f13fb7d93687eb22b63648f0ab6 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(5)
                timeout(time:1, unit:'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Implantação do backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('Testes de API') {
            steps {
                dir('api-test') {
                    git branch: 'main', credentialsId: 'ef3aa016-7687-4703-ad81-2262e79db02b', url: 'https://github.com/alessanderbotti/tasks-api-test.git'
                    sh 'mvn test'
                }
            }
        }
        stage ('Implantação do frontend') {
            steps {
                dir('frontend') {
                    git credentialsId: 'ef3aa016-7687-4703-ad81-2262e79db02b', url: 'https://github.com/alessanderbotti/tasks-frontend.git'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Testes funcionais') {
            steps {
                dir('functional-test') {
                    git branch: 'main', credentialsId: 'ef3aa016-7687-4703-ad81-2262e79db02b', url: 'https://github.com/alessanderbotti/tasks-functional-test.git'
                    sh 'mvn test'
                }
            }
        }
        stage ('Implantação em produção') {
            steps {
                sh 'docker-compose build'
                sh 'docker-compose up -d'
            }
        }
        stage ('Health Check') {
            steps {
                sleep(5)
                dir('functional-test') {
                    sh 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war', onlyIfSuccessful: true
        }
        unsuccessful {
            emailext attachLog: true, body: 'See the attached log below.', subject: 'Build $BUILD_NUMBER has failed.', to: 'alessanderbotti+jenkins@gmail.com'
        }
        fixed {
            emailext attachLog: true, body: 'See the attached log below.', subject: 'Build is fine!', to: 'alessanderbotti+jenkins@gmail.com'
        }
    }
}