pipeline {
  agent {
    kubernetes {
      label 'mypodtemplate-maven-3.5.4'
      containerTemplate {
        name 'maven'
        image 'maven:3.5.4-jdk-8-alpine'
        ttyEnabled true
        command 'cat'
      }
    }
  }
  stages {
    stage('Work') {
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
        stage('jenkins-demo') {
          steps {
            ws(dir: 'jenkins-demo') {
              git(url: 'https://github.com/MarkEWaite/jenkins-demo', branch: 'master')
              sh 'java -version'
              ws(dir: 'sample_maven') {
                sh 'echo current directory contents && pwd && ls'
                sh 'echo parent directory contents && ls ..'
                sh 'mvn --version'
                sh 'mvn clean install'
              }
            }
          }
        }
      }
    }
  }
}
