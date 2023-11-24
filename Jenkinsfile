pipeline {
    agent any

    stages {
        stage('Database Maintenance!') {
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
                        echo "hello"
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
                            sh """
                                curl -X POST \
                                -H "Authorization: token $GITHUB_TOKEN" \
                                -H "Accept: application/vnd.github.v3+json" \
                                -d '${payload as groovy.json.JsonOutput}' \
                                https://api.github.com/repos/Luckvill/Test/hooks
                            """
                        } else {
                            echo "its not me"
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
                currentBuild.result = 'SUCCESS'
                echo 'Database maintenance successful!'
            }
        }
        failure {
            script {
                currentBuild.result = 'FAILURE'
                echo 'Database maintenance failed!'
            }
        }
    }
}

