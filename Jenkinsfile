pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', credentialsId: 'log-in-to-github', url: 'git@github.com:noaeisnkut/my_Jenkins_project.git'
      }
    }
    stage('Skip if Jenkins commit') {
      steps {
        script {
          // On Windows nodes
          def lastMsg   = bat(returnStdout: true, script: 'git log -1 --pretty=%B').trim()
          def lastEmail = bat(returnStdout: true, script: 'git log -1 --pretty=%ae').trim()
          if (lastEmail.equalsIgnoreCase('jenkins@my-company.com') || lastMsg.contains('[skip ci]') || lastMsg.contains('[ci skip]')) {
            echo "Skipping build triggered by Jenkins' own version-bump commit."
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
        }
      }
    }

    stage('Commit and Push Version') {
      steps {
        script {
          // Commit only if there are staged changes, and only then push
          bat '''
          git config user.email "jenkins@my-company.com"
          git config user.name "Jenkins"
          git add Version.text
          setlocal enabledelayedexpansion
          git diff --cached --quiet && (echo No changes to commit & set COMMITTED=0) || (git commit -m "[skip ci] Update version to %NEW_VERSION%" & set COMMITTED=1)
          if "!COMMITTED!"=="1" git push origin main
          '''
        }
      }
    }

    stage('Build Project') { steps { bat "echo Building project..." } }

    stage('Docker Build & Push') {
      steps {
        bat "docker build -t noa10203040/simple_image:%NEW_VERSION% ."
        bat "docker push noa10203040/simple_image:%NEW_VERSION%"
      }
    }
  }
}
