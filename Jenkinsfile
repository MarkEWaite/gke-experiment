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
        stage('simple-java-maven-app') {
          steps {
            ws(dir: 'simple-java-maven-app') {
              git(url: 'https://github.com/jenkins-docs/simple-java-maven-app', branch: 'master')
              sh 'java -version'
              sh 'mvn --version'
              sh 'mvn clean install'
            }
          }
        }
        stage('jenkins-demo') {
          steps {
            ws(dir: 'jenkins-demo') {
              git(url: 'https://github.com/MarkEWaite/jenkins-demo', branch: 'master')
              sh 'java -version'
              sh 'mvn --version'
              ws(dir: 'sample_maven') {
                sh 'mvn clean install'
              }
            }
          }
        }
      }
    }
  }
}
