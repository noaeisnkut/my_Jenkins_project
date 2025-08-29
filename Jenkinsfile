pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main',
            credentialsId: 'log-in-to-github',
            url: 'git@github.com:noaeisnkut/my_Jenkins_project.git'
      }
    }

    stage('Skip if Jenkins commit') {
      steps {
        script {
          def lastMsg   = bat(returnStdout: true, script: '@echo off && git log -1 --pretty=%%B').trim()
          def lastEmail = bat(returnStdout: true, script: '@echo off && git log -1 --pretty=%%ae').trim()

          echo "Last commit message: ${lastMsg}"
          echo "Last commit author: ${lastEmail}"

          if (lastEmail.equalsIgnoreCase('jenkins@my-company.com') ||
              lastMsg.contains('[skip ci]') ||
              lastMsg.contains('[ci skip]')) {
            echo "Skipping build triggered by Jenkins' own commit."
            currentBuild.result = 'NOT_BUILT'
            error('Stop pipeline')
          }
        }
      }
    }

    stage('Update Version') {
      steps {
        script {
          def currentVersion = readFile("Version.text").trim()
          def parts = currentVersion.split('\\.')
          def major = parts[0].toInteger()
          def minor = parts[1].toInteger()
          def patch = parts[2].toInteger() + 1
          def newVersion = "${major}.${minor}.${patch}"
          writeFile file: "Version.text", text: newVersion
          env.NEW_VERSION = newVersion
          echo "Updated version to: ${env.NEW_VERSION}"
        }
      }
    }

    stage('Commit and Push Version') {
      steps {
        script {
          bat """
            @echo off
            git config user.email "jenkins@my-company.com"
            git config user.name "Jenkins"
            git add Version.text
            git diff --cached --quiet
            if errorlevel 1 (
              git commit -m "[skip ci] Update version to ${env.NEW_VERSION}"
              git push origin main
            ) else (
              echo No changes to commit
            )
          """
        }
      }
    }

    stage('Build Project') {
      steps {
        bat "echo Building project..."
      }
    }

    stage('Docker Build & Push') {
      steps {
        bat "docker build -t noa10203040/simple_image:${env.NEW_VERSION} ."
        bat "docker push noa10203040/simple_image:${env.NEW_VERSION}"
      }
    }
  }

