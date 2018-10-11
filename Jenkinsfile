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
        stage('simple-java-maven-app') {
          steps {
            ws(dir: 'simple-java-maven-app') {
              git(url: 'https://github.com/jenkins-docs/simple-java-maven-app', branch: 'master')
              sh 'mvn clean install'
            }
          }
        }
        stage('jenkins-demo') {
          steps {
            ws(dir: 'jenkins-demo') {
              git(url: 'https://github.com/MarkEWaite/jenkins-demo', branch: 'master')
            }
            ws(dir: 'jenkins-demo/sample_maven') {
              sh 'mvn clean install'
            }
          }
        }
      }
    }
  }
  post {
    always { 
      ws(dir: 'simple-java-maven-app') {
        junit '**/*.xml'
      }
      ws(dir: 'jenkins-demo') {
        junit '**/*.xml'
      }
    }
  }
}
