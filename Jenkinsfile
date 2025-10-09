node 
// /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.9.11/bin

{
    def mavenHome =tool name: "maven 3.9.11"
stage('git checkout'){

git branch: 'development', url: 'https://github.com/Puneeth-devopsonline/maven-webapplication-project-kkfunda.git'

}
stage('mvn build'){
  sh   "${mavenHome}/bin/mvn clean package"
}

stage('SQ Report'){
  sh   "${mavenHome}/bin/mvn sonar:sonar"
}
stage('deploy  to nexus'){
  sh   "${mavenHome}/bin/mvn deploy"
}
stage('deploy to tomcat') {
  echo "Deploying WAR file using curl command ..."
  sh """
    curl -u balu:password \
      --upload-file /var/lib/jenkins/workspace/pipeline-job/target/maven-web-application.war \
      "http://54.196.194.227:8085/manager/text/deploy?path=/maven-web-application&update=true"
  """


     
}
}//node
