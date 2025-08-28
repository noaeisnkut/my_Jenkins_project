pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    stages {
        stage('Check Author') {
            steps {
                script {
                    def author = bat(script: 'git log -1 --pretty=format:%an', returnStdout: true).trim()
                    if (author == 'Jenkins') {
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
                    bat 'git config user.email "jenkins@my-company.com"'
                    bat 'git config user.name "Jenkins"'
                    bat 'git add Version.text'
                    bat 'git commit -m "Jenkins auto version update to ${env.NEW_VERSION}" || echo No changes to commit'
                    bat 'git push origin main'
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
                script {
                    bat "docker build -t noa10203040/simple_image:${env.NEW_VERSION} ."
                    bat "docker push noa10203040/simple_image:${env.NEW_VERSION}"
                }
            }
        }
    }
}

