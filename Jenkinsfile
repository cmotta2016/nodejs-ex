timestamps {
    node('nodejsoci') {
        stage('Checkout'){
            checkout scm
            //checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/cmotta2016/nodejs-ex.git']]])
        }
        stage('Compile'){
            sh 'npm install'
        }
        stage ('Test'){
            sh 'npm test'
        }
        stage('Build'){
            sh 's2i build . openshift/nodejs-010-centos7 cmotta2016/k8s-nodejs --loglevel 1 --network host'
        }
        stage('Push Image'){
            withCredentials([usernamePassword(credentialsId: 'docker-io', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
            sh '''
            docker login -u "$DOCKER_USERNAME" -p "$DOCKER_PASSWORD"
            docker push cmotta2016/k8s-nodejs
            '''
            }
        }
        stage('Deploy'){
            sh 'kubectl apply -f k8s-nodejs.yml'
        }
    }
}
