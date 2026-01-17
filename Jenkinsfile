pipeline {
    agent any
    stages {
        stage ('test') {
            bat './gradlew test'
        }
        stage ('build') {
            steps {
                bat './gradlew build'
            }
        }
    }
}