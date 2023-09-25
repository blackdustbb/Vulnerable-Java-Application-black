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
                    ls
                '''
            }
            post {
                success {
                    archiveArtifacts 'dependency-check.txt'
                    // Send the Dependency-Check scan report to DefectDojo
                    sh '''
                        curl -X POST 'http://localhost:8081/api/v2/reimport-scan/' \
                        -H 'accept: application/json' \
                        -H 'Authorization: Token 3acbcf28101e0c357196bd9e861df0c7ae0dc46e' \
                        -F 'test=1' \
                        -F 'type=application/json' \
                        -F 'scan_type=Dependency Check Scan' \
                        -F 'File=@dependency-check-report.html' \
                        -F 'tags=test'
                    '''
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
                  zaproxy -cmd -quickurl http://localhost:1337 -quickprogress -quickout /opt/Scan_Report_ZAP.html
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

            // Publish HTML report
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: '/opt', // Change this to the correct directory
                reportFiles: 'Scan_Report_ZAP.html',    // Change this to the correct report file
                reportName: 'ZAP Report',
                reportTitles: 'ZAP Report'
            ])

            // Send email notifications
            emailext(
                subject: "Build ${currentBuild.result}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """
                <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                <p>Git Secrets Report: <a href="${env.BUILD_URL}/artifact/secrets.txt">secrets.txt</a></p>
                <p>SAST Output: <a href="${env.BUILD_URL}/artifact/SAST_output.txt">SAST_output.txt</a></p>
                <p>ZAP Report: <a href="${env.BUILD_URL}/artifact/output_ZAP.html">output_ZAP.html</a></p>
                """,
                to: "bluedustbb@gmail.com", // Add the recipient email address here
                attachLog: true, // Attach build log
                compressLog: true // Compress build log
            )
        }
    }
}
