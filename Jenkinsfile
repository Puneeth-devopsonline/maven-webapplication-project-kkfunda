pipeline {
    agent any
    tools {
        maven "maven 3.9.11"
    }
    stages {
        stage('git checkout') {
            steps {
                notifyBuild('STARTED')
                git branch: 'development', url: 'https://github.com/Puneeth-devopsonline/maven-webapplication-project-kkfunda.git'
            }
        }
        stage('build') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('SQ Report') {
            steps {
                sh "mvn sonar:sonar"
            }
        }
        stage('Uploadinto Nexus') {
            steps {
                sh "mvn deploy"
            }
        }
        stage('Deployment Approval') {
            steps {
                script {
                    // Send Slack notification asking for approval
                    slackSend(color: '#FFFF00', message: "Deployment ready for approval. Please review and approve or abort.", channel: '#jio-project')
                    try {
                        timeout(time: 30, unit: 'MINUTES') {
                            input message: 'Approve deployment?', ok: 'Deploy', submitter: 'team-lead'
                        }
                    } catch (err) {
                        def user = err.getCauses()[0]?.getUser()
                        if ('SYSTEM' == user?.toString()) {
                            slackSend(color: '#FFA500', message: "Deployment approval timed out after 30 minutes.", channel: '#jio-project')
                        } else {
                            slackSend(color: '#FF0000', message: "Deployment aborted by ${user}.", channel: '#jio-project')
                        }
                        error("Deployment not approved. Aborting pipeline.")
                    }
                }
            }
        }
        stage('tomcat deploy') {
            steps {
                sh """
                    curl -u balu:password \
                    --upload-file /var/lib/jenkins/workspace/MBPL/target/maven-web-application.war \
                    "http://18.212.234.235:8085/manager/text/deploy?path=/maven-web-application&update=true"
                """
            }
        }
    }
    post {
        success {
            script {
                notifyBuild(currentBuild.result)
            }
        }
        failure {
            script {
                notifyBuild(currentBuild.result)
            }
        }
    }
}

// Notification method
def notifyBuild(String buildStatus = 'STARTED') {
    buildStatus = buildStatus ?: 'SUCCESS'

    def colorCode
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    switch (buildStatus) {
        case 'STARTED':
            colorCode = '#FFFF00' // Yellow
            break
        case 'SUCCESS':
            colorCode = '#00FF00' // Green
            break
        default:
            colorCode = '#FF0000' // Red
    }

    slackSend(color: colorCode, message: summary, channel: '#jio-project')
    slackSend(color: colorCode, message: summary, channel: '#jio-dev')
}
