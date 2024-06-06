pipeline {
    agent any
    stages {
        stage ('Build the backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
    }
}