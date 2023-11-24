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
                        echo "hello"
                        // Check if the webhook exists
                        if (!existingWebhook.contains("$URL")) {
                            echo "its me"
                            createWebhook(${GITHUB_TOKEN}, ${URL})
                            echo ":D"
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

def createWebhook(token, webhookURL) {
    def payload = [
        name: 'web',
        active: true,
        events: ['pull_request'],
        config: [
            url: webhookURL,
            content_type: 'json',
            secret: '111111'
        ]
    ]
    echo "1"
    def apiUrl = 'https://api.github.com/repos/Luckvill/Test/hooks'
    def headers = [
        'Authorization': "token $token",
        'Accept': 'application/vnd.github.v3+json'
    ]
    echo "2"
    def response = httpRequest(
        httpMode: 'POST',
        url: apiUrl,
        requestBody: groovy.json.JsonOutput.toJson(payload),
        contentType: 'APPLICATION_JSON',
        headers: headers
    )
    echo "3"
    if (response.status == 201) {
        echo 'Webhook creado exitosamente.'
        echo "4"
    } else {
        error "Error al crear el webhook: ${response.status} - ${response.content}"
        echo "4 mal"
    }
}
