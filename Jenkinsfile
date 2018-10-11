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
          sh 'mvn --version'
        }
      }
    }
  }
}
