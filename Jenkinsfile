#!groovy

@Library('github.com/teecke/jenkins-pipeline-library@v3.4.1') _

// Initialize global config
cfg = jplConfig('dide-cypress', 'docker', '', [email: env.CITEECKE_NOTIFY_EMAIL_TARGETS])

def publishDockerImage() {
    nextReleaseNumber = sh (script: "kd get-next-release-number .", returnStdout: true).trim().substring(1)
    docker.withRegistry("https://registry.hub.docker.com", 'teeckebot-docker-credentials') {
        def customImage = docker.build("kairops/dide-cypress:${nextReleaseNumber}", "--pull --no-cache .")
        customImage.push()
        customImage.push('latest')
    }
}

pipeline {
    agent { label 'docker' }

    stages {
        stage ('Initialize') {
            steps  {
                jplStart(cfg)
            }
        }
        stage ('Bash linter') {
            steps {
                script {
                    sh 'devcontrol run-bash-linter'
                }
            }
        }
        stage ('Build') {
            agent { label 'docker' }
            steps {
                script {
                    docker.build('kairops/dide-cypress:test', '--pull --no-cache .')
                }
            }
        }
        stage ('Make release') {
            when { branch 'release/new' }
            steps {
                publishDockerImage()
                jplMakeRelease(cfg, true)
            }
        }
    }

    post {
        always {
            jplPostBuild(cfg)
        }
    }

    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(artifactNumToKeepStr: '20',artifactDaysToKeepStr: '30'))
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
    }
}