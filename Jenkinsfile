pipeline {
    agent any

    stages {
        stage('Checkout') {
            when {
                not {
                    expression {
                        def commitMessage = bat(returnStdout: true, script: 'git log -1 --pretty=format:%B').trim()
                        return commitMessage.contains('[ci skip]')
                    }
                }
            }
            steps {
                git branch: 'main', credentialsId: 'log-in-to-github', url: 'git@github.com:noaeisnkut/my_Jenkins_project.git'
            }
        }
        stage('Update Version') {
            when {
                not {
                    expression {
                        def commitMessage = bat(returnStdout: true, script: 'git log -1 --pretty=format:%B').trim()
                        return commitMessage.contains('[ci skip]')
                    }
                }
            }
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
            when {
                not {
                    expression {
                        def commitMessage = bat(returnStdout: true, script: 'git log -1 --pretty=format:%B').trim()
                        return commitMessage.contains('[ci skip]')
                    }
                }
            }
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
            when {
                not {
                    expression {
                        def commitMessage = bat(returnStdout: true, script: 'git log -1 --pretty=format:%B').trim()
                        return commitMessage.contains('[ci skip]')
                    }
                }
            }
            steps {
                bat 'echo Building project...'
            }
        }
        stage('Docker Build & Push') {
            when {
                not {
                    expression {
                        def commitMessage = bat(returnStdout: true, script: 'git log -1 --pretty=format:%B').trim()
                        return commitMessage.contains('[ci skip]')
                    }
                }
            }
            steps {
                script {
                    bat "docker build -t noa10203040/simple_image:${env.NEW_VERSION} ."
                    bat "docker push noa10203040/simple_image:${env.NEW_VERSION}"
                }
            }
        }
    }
}

