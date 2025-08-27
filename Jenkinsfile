pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'log-in-to-github', url: 'git@github.com:noaeisnkut/my_Jenkins_project.git'
            }
        }

        stage('Check Commit for CI Skip') {
            steps {
                script {
                    def commitMessage = bat(returnStdout: true, script: 'git log -1 --pretty=%B').trim()
                    if (commitMessage.contains('[ci skip]')) {
                        echo "Commit message contains '[ci skip]', skipping this build to prevent a build loop."
                        error('Build aborted by Jenkinsfile due to [ci skip] tag.')
                    }
                }
            }
        }
        
        stage('Update Version') {
            steps {
                script {
                    def currentVersion = readFile('Version.text').trim()
                    def parts = currentVersion.split('\\.')
                    def major = parts[0].toInteger()
                    def minor = parts[1].toInteger()
                    def patch = parts[2].toInteger()
                    patch += 1
                    def newVersion = "${major}.${minor}.${patch}"
                    writeFile file: 'Version.text', text: newVersion
                    env.NEW_VERSION = newVersion
                }
            }
        }
        stage('Commit and Push Version') {
            steps {
                script {
                    bat 'git config user.email "jenkins@my-company.com"'
                    bat 'git config user.name "Jenkins"'
                    bat 'git add Version.text'
                    bat "git commit -m \"[ci skip] Update version to ${env.NEW_VERSION}\" || echo No changes to commit"
                    bat 'git push origin main'
                }
            }
        }
        stage('Build Project') {
            steps {
                bat 'echo Building project...'
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    bat "docker build -t noa10203040/simple_image:${env.NEW_VERSION} ."
                    bat "docker push noa10203040/simple_image:${env.NEW_VERSION}"
                }
            }
        }
    }
}




