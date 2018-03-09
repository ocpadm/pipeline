pipeline {
  agent any
  stages {
    stage('hallo') {
      steps {
        script {
          echo ""
        }
        
      }
    }
    stage('confirmRelease') {
      steps {
        script {
          timeout(time: 5, unit: 'MINUTES') {
            env.RELEASE_SCOPE = input message: 'User input 				required', id: release, ok: 'Release!',
            parameters: [choice(name: 'RELEASE_SCOPE', choices: 		'SIT\nUAT\nPROD', description: 'What is the release scope?')]
          }
        }
        
      }
    }
  }
}