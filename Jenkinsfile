pipeline {
  agent any
  environment {
    VERSION = "1.0"
  }
  stages {
    stage('checkout') {
      steps {
        git branch: 'main', credentialsId: 'log-in-to-github' , url: 'git@github.com:noaeisnkut/my_Jenkins_project.git'
      }
    }
    stage('update version') {
      steps {
        script {
          def currentVersion = readFile('version.text').trim()
          def newVersionNumber = currentVersion.toInteger() + 1
          writeFile file: 'version.text', text: "${newVersionNumber}"
          env.NEW_VERSION = newVersionNumber
        }
      }
    }
    stage('Commit and Push Version') {
      steps {
        script {
          sh 'git config user.email "jenkins@my-company.com"'
          sh 'git config user.name "Jenkins"'
          sh 'git add version.txt'
          sh "git commit -m '[ci skip] Increment version to ${env.NEW_VERSION}'"
          sh 'git push origin main'
        }
      }
    }
    stage ('build') {
      steps {
        sh 'echo "building project..."'
      }
    }
    stage('docker build & push to DOCKER HUB') {
      steps {
        script {
          def dockerImage = docker.build("noa10203040/simple_image:${env.NEW_VERSION}")
          docker.withRegistry('https://registry.hub.docker.com', 'login-to-dockerhub') {
            dockerImage.push()
          }
        }
      }
    }
  }
}  

  

