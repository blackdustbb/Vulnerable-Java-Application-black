pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    sh """
                        echo "PATH = ${PATH}"
                        echo "M2_HOME = ${M2_HOME}"
                    """
                }
            }
        }

        stage('Scan Code with git-secrets') {
            steps {
                sh '''
                     git secrets --scan -r /opt/Vulnerable-Java-Application 2>&1 | tee secrets.txt
                '''
            }
            post {
                success {
                    archiveArtifacts 'secrets.txt'
                }
            }
        }

        stage('Scan code with dependency-check') {
            steps {
                sh '''
                    /opt/dependency-check/bin/dependency-check.sh --project "test" --scan "/opt/Vulnerable-Java-Application" | tee dependency-check.txt
                '''
            }
            post {
                success {
                    archiveArtifacts 'dependency-check.txt'
                }
            }
        }

        stage('Static Application Security Testing') {
            steps {
                sh '''
                    snyk code test /opt/Vulnerable-Java-Application | tee SAST_output.txt
                '''
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Archive Dependency-Check Report') {
            steps {
                archiveArtifacts artifacts: 'dependency-check-report.html', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            // Archive the git-secrets results as a build artifact
            archiveArtifacts 'secrets.txt'
            archiveArtifacts 'SAST_output.txt'
        }
    }
}
