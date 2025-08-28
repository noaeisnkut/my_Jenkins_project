pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    stages {
        stage('Check Commit') {
            steps {
                script {
                    def author = powershell(returnStdout: true, script: 'git log -1 --pretty=format:%an').trim().toLowerCase()
                    def message = powershell(returnStdout: true, script: 'git log -1 --pretty=format:%s').trim()

                    echo "Last commit author: '${author}'"
                    echo "Last commit message: '${message}'"

                    if (author.contains("jenkins") || message.contains("Jenkins auto version update")) {
                        currentBuild.result = 'NOT_BUILT'
                        error("Skipping build for Jenkins commit")
                    }
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'log-in-to-github', url: 'git@github.com:noaeisnkut/my_Jenkins_project.git'
            }
        }

        stage('Update Version') {
            steps {
                script {
                    def currentVersion = readFile("Version.text").trim()
                    def parts = currentVersion.split('\\.')
                    def major = parts[0].toInteger()
                    def minor = parts[1].toInteger()
                    def patch = parts[2].toInteger()
                    patch += 1
                    def newVersion = "${major}.${minor}.${patch}"
                    writeFile file: "Version.text", text: newVersion
                    env.NEW_VERSION = newVersion
                }
            }
        }

        stage('Commit and Push Version') {
            steps {
                script {
                    powershell 'git config user.email "jenkins@my-company.com"'
                    powershell 'git config user.name "Jenkins"'
                    powershell 'git add Version.text'
                    powershell 'git commit -m "Jenkins auto version update to ${env.NEW_VERSION}" || echo No changes to commit'
                    powershell 'git push origin main'
                }
            }
        }

        stage('Build Project') {
            steps {
                powershell 'echo Building project...'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    powershell "docker build -t noa10203040/simple_image:${env.NEW_VERSION} ."
                    powershell "docker push noa10203040/simple_image:${env.NEW_VERSION}"
                }
            }
        }
    }
}

