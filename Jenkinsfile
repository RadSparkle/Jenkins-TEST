#!groovy
import groovy.json.JsonOutput

def mkdir = """
          rm -rf /home/jenkins/api || exit
          [[ ! -d "/home/jenkins/api" ]] && mkdir /home/jenkins/api
          """ as java.lang.Object

pipeline {
    agent any
//     parameters {
//         gitParameter branch: 'origin/$BRANCH_NAME', name: 'REVISION', type: 'PT_REVISION'
//     }
    environment {
        SLACK_CHANNEL = '#배포방'
        BUILD_USER_ID = "테스트유저"
        BUILD_USER = "테스트유저"
    }
    stages {
        stage('check build user') {
                steps {
                        script {
                            BUILD_USER_ID = "${env.BUILD_USER_ID}"
                            BUILD_USER = "${env.BUILD_USER}"
                        }
                    // Test out of wrap
                    echo "Build User ID: ${BUILD_USER_ID}"
                    echo "Build User: ${BUILD_USER}"
                }
        }
        stage('checkout') {
                    steps {
                        checkout(
                                [
                                        $class                           : 'GitSCM',
                                        branches                         : [[name: '$BRANCH_NAME']],
                                        doGenerateSubmoduleConfigurations: false,
                                        extensions                       : [],
                                        submoduleCfg                     : [],
                                        userRemoteConfigs                : [[credentialsId : 'Admin', name:'origin',url: 'https://github.com/RadSparkle/Jenkins-TEST.git']]
                                ]
                                )
                    }
        }
        stage('Start') {
                    steps {
                        slackSend (channel: SLACK_CHANNEL, color: '#FFFF00', message: "배포 시작: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) ${env.BUILD_USER}")
                           }
        }
        stage('Build & Release') {
                    steps {
                        script {
                            try {
                                sh ("cd ${JENKINS_HOME}/workspace/TEST")
                                sh ("chmod 755 ./gradlew")
                                sh ("./gradlew clean bootJar")
                                env.jarfile = sh (script: 'basename build/libs/*.jar .jar', returnStdout: true ).trim()
                                echo "set File ${env.jarfile}.jar"
                                sh ("ls -la")
                                sh ("whoami")
                                sh ("cp build/libs/*.jar /home/jenkins/api/${env.jarfile}.jar")
                            } catch (e) {
                                slackSend (channel: SLACK_CHANNEL, color: '#FF0000', message: "빌드 실패: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                            }
                        }
                    }
        }
    }
    post {
        success {
            slackSend (channel: SLACK_CHANNEL, color: '#00FF00', message: "배포 성공: '${env.BUILD_USER} 'Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (channel: SLACK_CHANNEL, color: '#FF0000', message: "배포 실패: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
}