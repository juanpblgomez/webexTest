pipeline {
    agent any

    environment {
        JIRA_BASE_URL = "https://santex.atlassian.net/browse/"
        WEBEX_ROOM_ID = "Y2lzY29zcGFyazovL3VybjpURUFNOnVzLXdlc3QtMl9yL1JPT00vNzA2MDk2MjAtM2FlMy0xMWVmLTkwMjEtMzFlNGRkYTk0Mzk1"
        WEBEX_ACCESS_TOKEN = "NDdkM2IyYTctNDYyOC00ZDkwLThlNTEtNzQxY2E5NTRkMjBjOGVjMWRkMTktM2Iz_P0A1_e60e91ca-26e5-462f-aa2b-364e3ee84ba2"
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                // Aquí puedes poner comandos para construir tu proyecto
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                // Aquí puedes poner comandos para probar tu proyecto
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                // Aquí puedes poner comandos para desplegar tu proyecto
            }
        }
    }

    post {
        success {
            notifyWebex()
        }
    }
}

def notifyWebex() {
    script {
        // Obtener los commits desde el último despliegue exitoso
        def commits = sh(
            script: """
            git log --no-merges --pretty=format:"%s" ${env.GIT_PREVIOUS_SUCCESSFUL_COMMIT}..${env.GIT_COMMIT}
            """,
            returnStdout: true
        ).trim()

        println("Commits desplegados:\n${commits}")

        // Armar el mensaje Markdown
        def message = "**Listado de tareas desplegadas:**\n"
        def jiraRegex = ~/(\[?([A-Z]+-\d+)\]?)/

        commits.split('\n').each { commitMessage ->
            // Extraer el número de ticket JIRA del mensaje del commit (asumiendo que sigue un patrón como "PROJ-1234")
            def jiraTicketMatcher = commitMessage =~ jiraRegex
            if (jiraTicketMatcher) {
                def jiraTicket = jiraTicketMatcher[0][2]
                message += "\n- [${commitMessage}](${env.JIRA_BASE_URL}${jiraTicket})"
            } else {
                message += "\n- ${commitMessage}"
            }
        }

        // Reemplazar saltos de línea por \\n para el formato JSON
        def jsonMessage = message.replace("\n", "\\n")

        // Enviar el mensaje a Webex usando curl
        sh """
        curl -X POST \
          https://webexapis.com/v1/messages \
          -H "Authorization: Bearer ${env.WEBEX_ACCESS_TOKEN}" \
          -H "Content-Type: application/json" \
          -d '{
                "roomId": "${env.WEBEX_ROOM_ID}",
                "markdown": "${jsonMessage}"
              }'
        """
    }
}
