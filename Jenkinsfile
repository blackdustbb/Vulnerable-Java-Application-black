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

        stage('Scan Code with Trufflehog') {
            steps {
                sh "trufflehog --json /opt/Vulnerable-Java-Application | tee trufflehog-output.txt"
            }
        }

         stage('Scan Code with git-secrets') {
            steps {
                sh 'cd /opt/Vulnerable-Java-Application'
                sh 'git-secrets --scan -r /opt/Vulnerable-Java-Application | tee git-secrets.txt'
            }
        }
