#!/user/bin/env groovy

def gv

pipeline {
    agent any

    tools {
        maven 'maven-3.9'
    }

    stages {
        stage('increment java app version') {
            steps {
                script {
                    echo '[LOG] Incrementing the application version'
                    sh "mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit"
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }

        stage('build java app') {
            steps {
                script {
                    echo '[LOG] Building the application'
                    sh 'mvn clean package'
                }
            }
        }

        stage('build and push docker image') {
            steps {
                script {
                    echo '[LOG] Building the docker image'
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "docker build -t foundryvault/demo-app:${IMAGE_NAME} ."
                        sh "echo  $PASS | docker login -u $USER --password-stdin"
                        sh "docker push foundryvault/demo-app:${IMAGE_NAME}"
                    }
                }
            }
        }

        stage('deploy java app') {
            steps {
                script {
                    echo '[LOG] Deploying the application'
                }
            }
        }
    }
}