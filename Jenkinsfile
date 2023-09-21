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

stage('Dynamic Application Security Testing') {
    steps {
        script {
            // Define the target URL and report file path
            def targetUrl = "http://localhost:1337" // Replace with your actual URL
            def reportFile = "/opt/output_ZAP.html"

            // Run ZAP and generate an HTML report
            sh "/opt/zaproxy/zap.sh -quickurl ${targetUrl} -quickprogress -exportreport ${reportFile} -exportreport.format html"

            // Check if the ZAP report file exists before archiving it
            if (fileExists(reportFile)) {
                archiveArtifacts artifacts: reportFile, allowEmptyArchive: true
            } else {
                error("ZAP HTML report not found at ${reportFile}")
            }
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
                archiveArtifacts artifacts: 'output_ZAP.html', allowEmptyArchive: true
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
