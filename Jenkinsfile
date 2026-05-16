pipeline {
    agent {
        node {
            label 'ubuntu-build-agent' // Restricts execution to your Ubuntu slave agent
        }
    }

    parameters {
        choice(
            name: 'CHOICES',
            choices: ['dev', 'test', 'prod'],
            description: 'Select environment'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs(deleteDirs: true, notFailBuild: true)
                echo "Selected Environment: ${params.CHOICES}"
                // In SCM mode, Jenkins handles checkout automatically, 
                // but adding an explicit checkout scm step guarantees proper workspace setup.
                checkout scm
            }
        }

        stage('GitLeaks Scan') {
            steps {
                echo '=== Running Secret Scanning ==='
                sh '''
                gitleaks detect \
                --source . \
                --report-format json \
                --report-path gitleaks-report.json || echo "GitLeaks scanning evaluation completed."
                '''
            }
        }

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

        stage('Maven Build') {
            when {
                expression {
                    params.CHOICES == 'prod'
                }
            }
            steps {
                input message: 'Approve before starting Maven Build?', ok: 'Proceed'
                sh 'mvn package -DskipTests'
            }
        }
    }

    post {
        always {
            echo "Pipeline complete. Status: ${currentBuild.currentResult}"
            script {
                if (fileExists('gitleaks-report.json')) {
                    archiveArtifacts artifacts: 'gitleaks-report.json', fingerprint: true
                }
            }
        }
        success {
            echo 'Pipeline executed successfully via SCM configuration!'
        }
        failure {
            echo 'Pipeline execution failed.'
        }
    }
}
