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
    stage('user') {
      steps {
        input(message: 'do you want to continue', id: 'continue')
      }
    }
  }
}