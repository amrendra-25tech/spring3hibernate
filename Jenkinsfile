pipeline {
    agent {
        node {
            label 'ubuntu-build-agent' // Explicitly runs this pipeline on your Ubuntu Slave node
        }
    }

    tools {
        jdk 'Java8'       // Matches the Name in your Global Tool Configuration pointing to Java 8
        maven 'Maven3'    // Matches the Name in your Global Tool Configuration pointing to Maven
    }

    parameters {
        choice(
            name: 'CHOICES',
            choices: ['dev', 'test', 'prod'],
            description: 'Select deployment environment'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                // Wipe the remote workspace first to clean out stale workspace files
                cleanWs(deleteDirs: true, notFailBuild: true)
                
                echo "Selected Environment: ${params.CHOICES}"
                
                // Automatically handles checking out your specific GitHub repository code
                checkout scm
            }
        }

        stage('GitLeaks Scan') {
            steps {
                echo '=== Running Secret Scanning ==='
                // Runs the scanner globally on the slave and safely dumps the results to a JSON file
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
                echo '=== Compiling Java Application Code ==='
                sh 'mvn clean compile'
            }
        }

        stage('Maven Test') {
            steps {
                echo '=== Running Application Unit Tests ==='
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
                // Interactive manual gate required for production branch safety
                input message: 'Approve before starting Maven Build?', ok: 'Proceed'
                
                echo '=== Packaging Production War Artifact ==='
                sh 'mvn package -DskipTests'
            }
        }
    }

    post {
        always {
            echo "Pipeline Run Finalized. Final Build Status: ${currentBuild.currentResult}"
            
            // Checks if the security report exists before archiving it to avoid throwing errors
            script {
                if (fileExists('gitleaks-report.json')) {
                    archiveArtifacts artifacts: 'gitleaks-report.json', fingerprint: true
                }
            }
        }
        success {
            echo 'Pipeline executed successfully on the remote distributed architecture!'
        }
        failure {
            echo 'Pipeline build failed. Please review the specific stage logs above.'
        }
    }
}
