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
            cd /opt/Vulnerable-Java-Application
            git-secrets --scan -r /opt/Vulnerable-Java-Application | tee git-secrets.txt
        '''
    }
}
        stage('Scan code with dependency-check'){
            steps {
                sh '''
                    cd /opt/dependency-check/dependency-check/bin
                    dependency-check.sh --project "test" --scan "/opt/Vulnerable-Java-Application" | tee dependency-check.txt
                '''
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
            // Archive the git-secrets results as a build artifact
                     archiveArtifacts 'git-secrets.txt'
                     archiveArtifacts 'dependency-check.txt'
            
        }
    }
}
