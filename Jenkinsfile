def notifySlack(STATUS, COLOR) {
    slackSend channel: 'C04A7QD5ELV',
            message: "",
            color: COLOR, tokenCredentialId: 'slack',
            teamDomain: 'project-yrv2240'
}

node{

    //init stage 시작
    stage("init"){
        sh "echo init"
    }

    //build stage 시작
    stage("build"){
        sh "echo 'start build' "
    }

    //deploy stage 시작
    stage("deploy"){
        sh "echo 'start deploy' "
    }
}
notifySlack("빌드 파이프라인 성공", "#00FF00")
