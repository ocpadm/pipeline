# pipeline
## dockerBuild
def call(String name, String version, int port) {
    echo "Building Docker Image ${name}:${version}"
    sh "docker rmi -f ${name}:${version}  || exit 0"
    sh "docker build --rm -t ${name}:${version} ."
    sh "docker rm -f ${name} || exit 0"
    sh "docker run -d  -p ${port}:8080 --name=${name} ${name}:${version}"

}
## dockerRun
def call(String name, String version, int port) {
    echo "Running Docker Container ${name}:${version} on port ${port}"
    sh "docker rm -f ${name} || exit 0"
    sh "docker run -d -p ${port}:8080 --name=${name} ${name}:${version}"
}
## downloadArtifact
def call(String group, String artifact, String version, String profile) {
	sh "curl --silent --insecure -o $workspace/${artifact}.war 'https://nexus.agilesolutions.com/service/local/artifact/maven/redirect?r=releases&g=${group}&a=${artifact}&v=${version}&p=war'"
	sh "curl --silent --insecure -o initial.cli https://mct/rest/profile/cli/${profile}"
}
## deploy profile
#!groovy
import groovy.json.JsonOutput
import groovy.json.JsonSlurper
import groovy.json.*
import java.text.SimpleDateFormat
def call(String id) {
	withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'jctd',usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
	    def url = "https://mct/rest/profile/deploy"
		def payload = JsonOutput.toJson(["profileId":"${id}", "jiraIssueKey":"${env.JIRA_ISSUE_KEY}"])
		def response = sh(returnStdout: true, script: "curl -s -X POST --header 'IdentityToken: ${PASSWORD}' -H \"Accept: application/json\" -H \"Content-type: application/json\" -X POST -d '${payload}' ${url}").trim()
		echo response.toString()
	}
}
## 
