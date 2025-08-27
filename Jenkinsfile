pipeline {
    agent any
    stages {
        stage('checkout') {
            steps {
                git branch: 'main', credentialsId: 'log-in-to-github', url: 'git@github.com:noaeisnkut/my_Jenkins_project.git'
            }
        }
        stage('update version') {
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
                    bat "git config user.email \"jenkins@my-company.com\""
                    bat "git config user.name \"Jenkins\""
                    bat "git add Version.text"
                    bat "git commit -m \"[ci skip] Update version to ${env.NEW_VERSION}\""
                    bat "git push origin main"
                }
            }
        }
        stage('build') {
            steps {
                bat 'echo building project...'
            }
        }
        stage('docker build & push to DOCKER HUB') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'login-to-dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        bat "docker build -t noa10203040/simple_image:${env.NEW_VERSION} ."
                        bat "docker login -u %DOCKER_USER% -p %DOCKER_PASSWORD%"
                        bat "docker push noa10203040/simple_image:${env.NEW_VERSION}"
                    }
                }
            }
        }
    }
}



