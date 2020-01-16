@Library('sharedlib') _
import src.dataxu.docker.*

def general_utils = new dataxu.general.utils()
def general_docker = new dataxu.docker.general(this)

pipeline {
    agent { label 'slave-docker' }
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }
    stages {
        stage('Prep Env') {
            steps {
                deleteDir()
                script {
                    general_docker.set_docker_env()
                    checkout scm
                }
            }
            post {
                always {
                    github_notify_status()
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh "docker build . -t ${env.ecr_uri}/github-release-notes:dev-${GIT_COMMIT}"
                }
            }
            post {
                always {
                    github_notify_status()
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    def VERSION = sh script: "docker run ${env.ecr_uri}/github-release-notes:dev-${GIT_COMMIT} npm -v github-release-notes", returnStdout: true
                    sh "docker tag ${env.ecr_uri}/github-release-notes:dev-${GIT_COMMIT} ${env.ecr_uri}/github-release-notes:rel-${VERSION}"
                    sh "docker tag ${env.ecr_uri}/github-release-notes:dev-${GIT_COMMIT} ${env.ecr_uri}/github-release-notes:latest"
                    sh "docker push ${env.ecr_uri}/github-release-notes:dev-${GIT_COMMIT}"
                }
            }
            post {
                always {
                    github_notify_status()
                }
            }
        }
    }
    post {
        always {
            script {
                general_docker.delete_image("${env.ecr_uri}/github-release-notes:${GIT_COMMIT}")
                github_notify_status(stage_name: 'Pipeline complete')
            }
        }
    }
}
