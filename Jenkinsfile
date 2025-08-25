pipeline {
  agent any
  environment {
    VERSION = "1.0"
  }
  stages {
    stage('checkout') {
      steps {
        git branch: 'main', credentialsId: 'log-in-to-github' , url: 
