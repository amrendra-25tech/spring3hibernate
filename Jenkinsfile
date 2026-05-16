pipeline {

    agent {
        node {
            label 'ubuntu-build-agent'
        }
    }

    tools {
        maven 'Maven3'
        jdk 'Java8'
    }

    stages {

        stage('GitLeaks Scan') {
            steps {

                sh '''
                gitleaks detect \
                --source . \
                --report-format json \
                --report-path gitleaks-report.json || true
                '''

            }
        }

        stage('Approval Before Build') {
            steps {

                input message: 'Do you want to continue with Maven Build and Test?',
                      ok: 'Proceed'

            }
        }

        stage('Parallel Build and Test') {

            parallel {

                stage('Maven Compile') {
                    steps {

                        sh 'mvn clean compile'

                    }
                }

                stage('Maven Test') {
                    steps {

                        sh 'mvn test'

                    }
                }
            }
        }
    }

    post {

        always {

            archiveArtifacts artifacts: 'gitleaks-report.json',
            allowEmptyArchive: true,
            fingerprint: true

        }
    }
}
