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

       
        stage('Handle Scan Results') {
    steps {
        script {
            def scanOutput = sh(script: 'git-secrets --scan -r /opt/Vulnerable-Java-Application ', returnStatus: true).trim()
            if (scanOutput.contains("✖")) {
                currentBuild.result = 'FAILURE'
                error("Secrets were found in the codebase.")
            } else {
                echo "No secrets were found."
            }
        }
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
