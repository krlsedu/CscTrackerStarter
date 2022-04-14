#!groovy

pipeline {
  agent none
  stages {
    stage('Build') {
      agent any
        tools {
                maven 'M3'
            }
      steps {
        sh 'mvn clean install package -DskipTests'
      }
    }
  }
}