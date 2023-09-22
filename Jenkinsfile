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
stage('Debug: List Workspace Contents') {
    steps {
        sh 'pwd'
    }
}
        stage('Dynamic Application Security Testing') {
            steps {
                sh '''
                  zaproxy -cmd -quickurl http://localhost:1337 -quickprogress -quickout /root/.jenkins/workspace/DevSecOps2/Report_new_ZAP.html

                '''
            }
            post {
                success {
                    archiveArtifacts '/root/.jenkins/workspace/DevSecOps2/Report_new_ZAP.html'
                }
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

       stage('Archive ZAP Report') {
           steps {
          archiveArtifacts artifacts: 'Report_new_ZAP.html', allowEmptyArchive: true
            }
    }
   }

    post {
        always {
            // Archive the git-secrets results as a build artifact
            archiveArtifacts 'secrets.txt'
            archiveArtifacts 'SAST_output.txt'

            // Publish HTML report
            //publishHTML([
                //allowMissing: false,
                //alwaysLinkToLastBuild: true,
                //keepAll: true,
                //reportDir: '/opt', // Change this to the correct directory
                //reportFiles: 'Report_new_ZAP.html',    // Change this to the correct report file
                //reportName: 'ZAP Report',
                //reportTitles: 'ZAP Report'
            //])
        }
    }
}
