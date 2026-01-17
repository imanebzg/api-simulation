pipeline {
    agent any

    tools {
        gradle 'Gradle' // Make sure Gradle is configured in Jenkins Global Tool Configuration
    }

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
                    sh './gradlew test'

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
                        sh './gradlew sonarqube'
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
                    sh './gradlew build -x test'

                    echo 'Generating Javadoc...'
                    sh './gradlew generateJavadoc'

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
                    sh './gradlew publish'
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