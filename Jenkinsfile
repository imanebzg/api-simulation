pipeline {
    agent any

    environment {
        MAVEN_REPO_URL = "${env.MAVEN_REPO_URL}"
        MAVEN_REPO_USERNAME = "${env.MAVEN_REPO_USERNAME}"
        MAVEN_REPO_PASSWORD = "${env.MAVEN_REPO_PASSWORD}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                script {
                    echo 'Running Unit Tests...'
                    // Use bat for Windows, sh for Linux
                    if (isUnix()) {
                        sh 'chmod +x gradlew'
                        sh './gradlew test'
                    } else {
                        bat 'gradlew.bat test'
                    }

                    echo 'Archiving Test Results...'
                    junit '**/build/test-results/test/*.xml'

                    echo 'Generating Cucumber Reports...'
                    cucumber buildStatus: 'UNSTABLE',
                            reportTitle: 'Cucumber Report',
                            fileIncludePattern: '**/*.json',
                            trendsLimit: 10,
                            classifications: [
                                [key: 'Browser', value: 'Chrome']
                            ]
                }
            }
        }

        stage('Code Analysis') {
            steps {
                script {
                    echo 'Running SonarQube Analysis...'
                    withSonarQubeEnv('SonarQube') {
                        if (isUnix()) {
                            sh './gradlew sonarqube'
                        } else {
                            bat 'gradlew.bat sonarqube'
                        }
                    }
                }
            }
        }

        stage('Code Quality') {
            steps {
                script {
                    echo 'Checking Quality Gate...'
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Building JAR file...'
                    if (isUnix()) {
                        sh './gradlew build -x test'
                    } else {
                        bat 'gradlew.bat build -x test'
                    }

                    echo 'Generating Javadoc...'
                    if (isUnix()) {
                        sh './gradlew generateJavadoc'
                    } else {
                        bat 'gradlew.bat generateJavadoc'
                    }

                    echo 'Archiving artifacts...'
                    archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
                    archiveArtifacts artifacts: '**/build/docs/javadoc/**/*', fingerprint: true
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying to MyMavenRepo...'
                    if (isUnix()) {
                        sh './gradlew publish'
                    } else {
                        bat 'gradlew.bat publish'
                    }
                }
            }
        }

        stage('Notification') {
            steps {
                script {
                    echo 'Sending success notification...'
                    emailext (
                        subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                        body: """<p>Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' has been deployed successfully!</p>
                                <p>Check console output at <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>""",
                        to: 'your-email@example.com',
                        mimeType: 'text/html'
                    )

                    // Slack notification (optional - requires Slack plugin)
                    // slackSend color: 'good', message: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' deployed successfully!"
                }
            }
        }
    }

    post {
        failure {
            script {
                echo 'Sending failure notification...'
                emailext (
                    subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    body: """<p>Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' has failed!</p>
                            <p>Check console output at <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>
                            <p>Failed at stage: ${env.STAGE_NAME}</p>""",
                    to: 'your-email@example.com',
                    mimeType: 'text/html'
                )

                // Slack notification (optional)
                // slackSend color: 'danger', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' - Stage: ${env.STAGE_NAME}"
            }
        }

        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}