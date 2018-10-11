pipeline {
  agent any
  stages {
    stage('Hello') {
      parallel {
        stage('Hello 1') {
          steps {
            sh 'echo "Shell hello world"'
          }
        }
        stage('Hello 2') {
          steps {
            echo 'Printing hello'
          }
        }
      }
    }
  }
}