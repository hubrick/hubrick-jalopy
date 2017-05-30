BUILD_TIMEOUT = 10
BUILD_TIMEOUT_UNIT = 'MINUTES' // valid values: 'SECONDS', 'MINUTES', 'HOURS'
MAVEN_TOOL_VERSION = 'maven-3.5.0' // tool name as defined in https://hubrick.ci.cloudbees.com/configureTools

pipeline {
    agent none

    options {
        buildDiscarder(logRotator(numToKeepStr:'3', daysToKeepStr:'60'))
        ansiColor('xterm')
        timestamps()
        timeout (time: BUILD_TIMEOUT, unit: BUILD_TIMEOUT_UNIT)
    }

    stages {

        stage('Build') {
            agent {
                label 'docker-ng'
            }
            steps {
                slackSend (channel: slackChannel(), color: '#00FF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                checkout scm
                script {
                    gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    withDockerRegistry([credentialsId: 'docker-registry-net-secret', url: 'https://docker.hubrick.net']) {
                        withMaven(maven: MAVEN_TOOL_VERSION, mavenSettingsConfig: 'maven-clean-settings-xml') {
                            sh "mvn -U clean package"
                        }
                    }
                }
            }
            post {
                always {
                    step ([$class: 'WsCleanup'])
                }
            }
        }
    }

    post {
        success {
            slackSend (channel: slackChannel(), color: '#00FF00', message: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (channel: slackChannel(), color: '#FF0000', message: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }

    }
}

def slackChannel() {
    return (env.BRANCH_NAME == 'master') ? '#jenkins-backend' : '#jenkins-pr'
}