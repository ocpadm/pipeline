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
## getConfig
def call(String profile) {
	sh "curl --silent --insecure -o initial.cli https://jctd/rest/profile/cli/${profile}"
}
## getFreePort
#!/usr/bin/groovy
def call() {
    sh (script: "ss -tln | awk 'NR > 1{gsub(/.*:/,\"\",\$4); print \$4}' | sort -un | awk -v n=9090 '\$0 < n {next}; \$0 == n {n++; next}; {exit}; END {print n}'", returnStdout:true)
}
## jiraFeedBack
def call(String comment) {
   withEnv(['JIRA_SITE=agilesolutions']) {
        		echo "Tagging JIRA deployment ticket ${env.JIRA_ISSUE_KEY}"
            	jiraAddComment idOrKey: env.JIRA_ISSUE_KEY , comment: "${comment}"
        	}
}
## jiraGetTicket
def call() {
             withEnv(['JIRA_SITE=agilesolutions']) {
	             def issue = jiraGetIssue idOrKey: env.JIRA_ISSUE_KEY, site: 'agilesolutions'
			     env.VERSION = issue.data.fields.get("summary")
			     echo "deploying version ${env.VERSION} through deployment ticket ${env.JIRA_ISSUE_KEY}"
			 }
}
## jiraInput
def call() {
   timeout(time: 5, unit: 'MINUTES') {
             env.JIRA_ISSUE_KEY = input message: 'Specify ISSUE KEY', parameters: [string(defaultValue: '', description: '', name: 'JIRA_ISSUE_KEY')]
             withEnv(['JIRA_SITE=agilesolutions']) {
	             def issue = jiraGetIssue idOrKey: env.JIRA_ISSUE_KEY, site: 'agilesolutions'
			     env.VERSION = issue.data.fields.get("summary")
			     echo "deploying version ${env.VERSION} through deployment ticket ${env.JIRA_ISSUE_KEY}"
			 }
    }
}
## jiraNotify
def call() {
        	withEnv(['JIRA_SITE=agilesolutions']) {
        		echo "Notify JIRA watchers ${env.JIRA_ISSUE_KEY}"
				def notify = [ subject: 'Update about ',
                    textBody: 'Update on deployment ticket ...',
                   	htmlBody: 'Update on deployment ticket ...',
                   	to: [ reporter: true,assignee: true]]
    			jiraNotifyIssue idOrKey: env.JIRA_ISSUE_KEY, notify: notify
            } // end with
}
## jiraTransition
def call() {
        	withEnv(['JIRA_SITE=agilesolutions']) {
        			echo "Complete JIRA deployment ticket ${env.JIRA_ISSUE_KEY}"
        			def transitionInput =
						[
        						transition: [id: '"${env.JIRA_ISSUE_KEY}
"']
    					]

				jiraTransitionIssue idOrKey:  env.JIRA_ISSUE_KEY, input: transitionInput

            } // end with
}
## mailFeedBack
def call(String mailRecipients, String message) {
            emailext (
      		subject: "${message}",
      		body: "${message}",
	        to: "${mailRecipients}",
      		replyTo: "admin@agilesolutions.com")
}
