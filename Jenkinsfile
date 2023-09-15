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
                sh 'cd /opt/Vulnerable-Java-Application'
                sh 'git-secrets --scan -r /opt/Vulnerable-Java-Application > git-secrets.txt'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }

       post {
        always {
            // Archive the Trufflehog results as a build artifact
            archiveArtifacts 'git-secrets.txt'
        }
    }
}
