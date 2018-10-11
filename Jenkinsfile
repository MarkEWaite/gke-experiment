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
      }
    }
  }
  post {
    always { 
      ws(dir: 'simple-java-maven-app') {
        junit '**/*.xml'
      }
    }
  }
}
