def issueKey = ''
def projectKey = 'DEMO'

node('jira_agent') {
    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref']
            ],
            causeString: 'Triggered on $ref',
            printContributedVariables: true,
            printPostContent: true,
            regexpFilterText: '$ref',
            regexpFilterExpression: 'refs/heads/main'
        )
    }
    stage('Login to Jira') {
        withCredentials([usernamePassword(credentialsId: 'jira_cred', usernameVariable: 'JIRA_USERNAME', passwordVariable: 'JIRA_PASSWORD')]) {
            def response = sh(script: """
                curl -X GET -H 'Content-Type: application/json' \
                -u \$JIRA_USERNAME:\$JIRA_PASSWORD \
                http://34.89.195.65:8090/rest/api/2/myself
            """, returnStdout: true).trim()
            echo "Login to Jira response: $response"
            if (!response.contains('"active":true')) {
                error "Failed to log in to Jira"
            }
        }
    }

    stage('Create Jira Task') {
        withCredentials([usernamePassword(credentialsId: 'jira_cred', usernameVariable: 'JIRA_USERNAME', passwordVariable: 'JIRA_PASSWORD')]) {
            def response = sh(script: """
                curl -X POST -H 'Content-Type: application/json' \
                -u \$JIRA_USERNAME:\$JIRA_PASSWORD \
                -d '{
                    "fields": {
                        "project": {"key": "${projectKey}"},
                        "summary": "New task created by Jenkins pipeline",
                        "description": "This task was created by the Jenkins pipeline.",
                        "issuetype": {"name": "Task"}
                    }
                }' \
                http://34.89.195.65:8090/rest/api/2/issue/
            """, returnStdout: true).trim()
            echo "Create Jira task response: $response"
            def jsonResponse = readJSON text: response
            issueKey = jsonResponse.key
            echo "Created Jira issue with key: $issueKey"
        }
    }

    stage('Move Task to Done') {
        withCredentials([usernamePassword(credentialsId: 'jira_cred', usernameVariable: 'JIRA_USERNAME', passwordVariable: 'JIRA_PASSWORD')]) {
            def transitionsResponse = sh(script: """
                curl -X GET -H 'Content-Type: application/json' \
                -u \$JIRA_USERNAME:\$JIRA_PASSWORD \
                http://34.89.195.65:8090/rest/api/2/issue/$issueKey/transitions
            """, returnStdout: true).trim()
            echo "Available transitions: $transitionsResponse"

            def transitions = readJSON text: transitionsResponse
            def transitionId = transitions.transitions.find { it.name == 'Done' }?.id
            if (!transitionId) {
                error "Transition to 'Done' not found"
            }

            def response = sh(script: """
                curl -X POST -H 'Content-Type: application/json' \
                -u \$JIRA_USERNAME:\$JIRA_PASSWORD \
                -d '{ "transition": { "id": "$transitionId" } }' \
                http://34.89.195.65:8090/rest/api/2/issue/$issueKey/transitions
            """, returnStdout: true).trim()
            echo "Move Jira issue to Done response: $response"
        }
    }
}

