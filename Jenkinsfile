pipeline {
  agent any
  stages {
    stage('Hello') {
      parallel {
        stage('Hello 1') {
          steps {
            sh 'echo "hello shell world"'
          }
        }
        stage('Hello 2') {
          steps {
            echo 'hello pipeline world'
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
