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
    stage('Maven') {
      steps {
        ws(dir: 'simple-java-maven-app') {
          git(url: 'https://github.com/jenkins-docs/simple-java-maven-app', branch: 'master')
        }

        withMaven(publisherStrategy: 'IMPLICIT') {
          sh 'mvn --version'
        }

      }
    }
  }
}