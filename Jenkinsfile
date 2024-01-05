node{
    
    def mavenHome=tool name:'maven3.9.6'

    echo "job Name is: ${env.JOB_NAME}"
    echo "Node name is : ${env.NODE_NAME}"
    echo "Jenkins Home is : ${env.Jenkins_Home}"
    echo "Jenkins URL is : ${env.Jenkins_URL}"

    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([pollSCM('* * * * *')])])
   try{
    stage('checkoutcode'){
        sendSlackNotifications("STARTED")
    git branch: 'development', credentialsId: 'a73f289f-71ec-4cf6-97da-85cf43ffc288', url: 'https://github.com/devops-oct2023blr/maven-web-application.git'
   }
   stage('build'){
    sh "${mavenHome}/bin/mvn clean package"
   }
   stage('ExcuteSonarQubeReport'){
    sh "${mavenHome}/bin/mvn clean sonar:sonar"
   }
   stage('UploadArtfactintoNexus'){
    sh "${mavenHome}/bin/mvn clean deploy"
   }
   stage('DeployappIntoTomcatServer'){
    sshagent(['cd79945a-35b5-43da-bee9-847a94df1bd1']) {
    sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@172.31.22.225:/opt/apache-tomcat-9.0.83/webapps/"
   }
   }
   }
   catch(e){
    currentBuild.result = "FAILURE"
     throw e
   }
   finally{
    sendSlackNotifications(currentBuild.result)
   }


}
//node pipeline closing


def sendSlackNotifications(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    colorName = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    colorName = 'GREEN'
    colorCode = '#00FF00'
  } else {
    colorName = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
