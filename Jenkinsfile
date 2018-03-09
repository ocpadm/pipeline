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
    stage('releaseconfirm') {
      steps {
        input(ok: 'Release!', message: 'User input required', id: 'release', submitterParameter: '[choice(name: \'RELEASE_SCOPE\', choices: \'SIT\\nUAT\\nPROD\', description: \'What is the release scope?\')]')
      }
    }
  }
}