node {
    // Set job properties and triggers outside try-catch, early in the pipeline run
    properties([
        pipelineTriggers([
           // pollSCM(' * * * * * '),   // Poll SCM every  minute
           //  cron(' * * * * * '),        // Build periodically if needed
            githubPush()            // GitHub webhook trigger if needed
        ])
    ])

    echo "git branch name: ${env.BRANCH_NAME}"
    echo "build number is: ${env.BUILD_NUMBER}"
    echo "node name is: ${env.NODE_NAME}"

    def mavenHome = tool name: "maven 3.9.11"

    try {
        notifyBuild('STARTED')

        stage('git checkout') {
            git branch: 'development', url: 'https://github.com/Puneeth-devopsonline/maven-webapplication-project-kkfunda.git'
        }
        stage('Maven Build') {
            sh "${mavenHome}/bin/mvn clean package"
        }
        stage('SQ Report') {
            sh "${mavenHome}/bin/mvn sonar:sonar"
        }
        stage('Deploy into Nexus') {
            sh "${mavenHome}/bin/mvn deploy"
        }
        stage('Deploy to Tomcat') {
            echo "Deploying WAR file using curl..."
            sh """
                curl -u balu:password \
                --upload-file /var/lib/jenkins/workspace/pipeline-job/target/maven-web-application.war \
                "http://54.196.194.227:8085/manager/text/deploy?path=/maven-web-application&update=true"
            """
        }
    } catch (e) {
        currentBuild.result = "FAILED"
        throw e
    } finally {
        notifyBuild(currentBuild.result)
    }
}

def notifyBuild(String buildStatus = 'STARTED') {
    buildStatus = buildStatus ?: 'SUCCESS'
    def color = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: Jobkkdevops '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    if (buildStatus == 'STARTED') {
        color = 'BLUE'
        colorCode = '#2757F5'
    } else if (buildStatus == 'SUCCESS') {
        color = 'GREEN'
        colorCode = '#00FF00'
    }

    slackSend(color: colorCode, message: summary, channel: '#airtel-project' )
    slackSend(color: colorCode, message: summary, channel: '#airtel-dev'  )
}
