#!groovy

def mkdir = """
          rm -rf /home/jenkins/TEST || exit
          [[ ! -d "/home/jenkins/TEST" ]] && mkdir /home/jenkins/TEST
          """ as java.lang.Object

def notifySlack(STATUS, COLOR) {
//    def user = "${BUILD_USER}"
    if ( user != "null" ) {
        slackSend channel: 'C04A7QD5ELV',
                message: "[${STATUS}]",
                color: COLOR, tokenCredentialId: 'slack',
                teamDomain: 'Project'
    }
}

env.jarfile
pipeline {
    agent any
    parameters {
        gitParameter branch: 'origin/$BRANCH_NAME', name: 'REVISION', type: 'PT_REVISION'
    }
    environment {
        BUILD_USER_ID = ""
        BUILD_USER = ""
        BUILD_USER_EMAIL = ""
    }
    stages {
//        stage('check build user') {
//            steps {
//                wrap([$class: 'BuildUser']) {
//                    script {
//                        BUILD_USER_ID = "${env.BUILD_USER_ID}"
//                        BUILD_USER = "${env.BUILD_USER}"
//                        BUILD_USER_EMAIL = "${env.BUILD_USER_EMAIL}"
//                    }
//                }
//                // Test out of wrap
//                echo "Build User ID: ${BUILD_USER_ID}"
//                echo "Build User: ${BUILD_USER}"
//                echo "Build User Email: ${BUILD_USER_EMAIL}"
//            }
//        }
        stage('checkout revision') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: "${params.REVISION}"]],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [],
                          gitTool: 'Default',
                          submoduleCfg: [],
                          userRemoteConfigs: [[url: 'https://github.com/RadSparkle/Jenkins-TEST.git']]
                ])
            }
        }
        stage('Test & Build') {
            steps {
                script {
                    try {
                        sh ("chmod 755 ./gradlew")
                        sh ("./gradlew clean build")
//                        env.warfile = sh (script: 'basename build/libs/*.war ROOT.war', returnStdout: true ).trim()
//                        echo "set File ${env.warfile}.war"
//                        sh ("ls -la")
                    } catch (e) {
                        notifySlack("java 빌드 실패", "#FF0000")
                    }
                }
            }
        }
        stage('SSH transfer') {
            steps([$class: 'BapSshPromotionPublisherPlugin']) {
                sshPublisher(
                        continueOnError: false, failOnError: true,
                        publishers: [
                                sshPublisherDesc(
                                        configName: "dev",
                                        verbose: true,
                                        transfers: [
                                                sshTransfer(
                                                        execCommand:mkdir
                                                ),
                                                sshTransfer(
                                                        sourceFiles: "build/libs/${env.warfile}.war",
                                                        removePrefix: "build/libs/",
                                                        remoteDirectory: "TEST/",
                                                        execCommand: "mv /home/jenkins/TEST/${env.warfile}.war /home/jenkins/TEST/ROOT.war"
                                                )
                                        ]
                                )
                        ]
                )
            }
        }
    }
    post {
        failure {
            notifySlack("빌드 파이프라인 실패", "#FF0000")
        }
        success {
            notifySlack("빌드 파이프라인 성공", "#00FF00")
        }
        always {
            cleanWs()
        }
    }
}