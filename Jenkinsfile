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
    }
}