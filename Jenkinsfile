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
             git secrets --scan -r /opt/Vulnerable-Java-Application > secrets.txt 2>/dev/null
        '''
    }
}
        stage('Scan code with dependency-check'){
            steps {
                sh '''
                    /opt/dependency-check/bin/dependency-check.sh --project "test" --scan "/opt/Vulnerable-Java-Application" | tee dependency-check.txt
                '''
            }
        }

        stage('Static Application Security Testing'){
            steps{
                sh '''
                    snyk code test /opt/Vulnerable-Java-Application | tee SAST_output.txt
                '''
            }
        }
        stage('Scan Code with git-secrets new') {
    steps {
        sh '''
             git secrets --scan -r /opt &> new.txt
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
                     archiveArtifacts 'secrets.txt'
                     archiveArtifacts 'dependency-check.txt'
                     archiveArtifacts 'SAST_output.txt'
                     archiveArtifacts 'new.txt'
            
        }
    }
}
