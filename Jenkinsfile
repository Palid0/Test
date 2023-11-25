pipeline {
    agent any

    stages {
        stage('Database Maintenance') {
            steps {
                script {
                    sh 'wget -O Employees.db https://github.com/Palid0/PROF-2023-Ejercicio4/blob/main/Employees.db'

                    // Data backup
                    sh 'sqlite3 Employees.db ".dump" > backup.sql'

                    // Update Data
                    sh 'rm Employees.db'
                    sh 'sqlite3 Employees.db < sqlite.sql'

                    // Restore previous data
                    sh 'sqlite3 Employees.db < backup.sql'
                    
                }
            }
        }

        stage('Create the Webhook') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'Borrar', variable: 'GITHUB_TOKEN')]) {
                        def existingWebhook = sh(
                            script: 'curl -s -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/repos/Luckvill/Test/hooks',
                            returnStdout: true).trim()
                        def URL = "http://" + sh(script: 'curl -s ifconfig.me', returnStdout: true).trim() + ":8080/ghprbhook/"
                        // Check if the webhook exists
                        if (!existingWebhook.contains("$URL")) {
                            def payload = [
                                name: 'web',
                                active: true,
                                events: ['pull_request'],
                                config: [
                                    url: URL,
                                    content_type: 'json'
                                ]
                            ]
                            def jsonPayload = groovy.json.JsonOutput.toJson(payload)
                            sh """
                                curl -X POST \
                                -H "Authorization: token $GITHUB_TOKEN" \
                                -H "Accept: application/vnd.github.v3+json" \
                                -d '${jsonPayload}' \
                                https://api.github.com/repos/Luckvill/Test/hooks
                            """
                        } else {
                            echo 'Webhook exists.'
                        }
                    }
                }
            }
        }
    }

   post {
        success {
            script {
                if (env.CHANGE_ID != null) {
                    def pullRequestSHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    def status = '{"state": "success", "description": "Pull Request build successfull", "context": "Jenkins"}'
                    withCredentials([string(credentialsId: 'Borrar', variable: 'GITHUB_TOKEN')]) {
                        sh """
                        curl -X POST \
                        -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -d '${status}' \
                        https://api.github.com/repos/Luckvill/Test/statuses/${pullRequestSHA}
                        """
                    }
                } else {
                    def commitSHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    withCredentials([string(credentialsId: 'Eric', variable: 'GITHUB_TOKEN')]) {
                        sh """
                        curl -X POST \
                        -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -d '{"state": "success", "description": "Database maintenance successful", "context": "Jenkins"}' \
                        https://api.github.com/repos/Palid0/Test/statuses/${commitSHA}
                        """
                    }
                }
            }
        }
        failure {
            script {
                if (env.CHANGE_ID != null) {
                    def pullRequestSHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    def status = '{"state": "failure", "description": "Pull Request build failed", "context": "Jenkins"}'
                    withCredentials([string(credentialsId: 'Borrar', variable: 'GITHUB_TOKEN')]) {
                        sh """
                        curl -X POST \
                        -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -d '${status}' \
                        https://api.github.com/repos/Luckvill/Test/statuses/${pullRequestSHA}
                        """
                    }
                } else {
                    def commitSHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    withCredentials([string(credentialsId: 'Eric', variable: 'GITHUB_TOKEN')]) {
                        sh """
                        curl -X POST \
                        -H "Authorization: token $GITHUB_TOKEN" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -d '{"state": "failure", "description": "Database maintenance failed", "context": "Jenkins"}' \
                        https://api.github.com/repos/Palid0/Test/statuses/${commitSHA}
                        """
                    }
                }
            }
        }
    }
}

