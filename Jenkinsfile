timestamps{
    def label="dev-backend"
    node('nodejs'){
        stage('checkout'){
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/cmotta2016/PipelineScriptsBackendProject.git']]])
        }
        stage('build'){
            sh 'npm install'
        }
        stage('teste'){
            sh 'npm test'
        }
        openshift.withCluster() {
            openshift.withProject('node-backend-1') {
                stage('cleanup') {
                    sh "oc delete all -l app=${label} -n node-backend-1"
                }
                stage('create') {
                    def PROJETO = 'node-backend-1'
                    sh "oc new-app --file=template-nodejs.yml --param=LABEL=dev-backend --param=NAME=dev-backend --namespace=node-backend-1"
                    sleep (30)
                }
                stage('BuildConfig') {
                    sh 'oc delete buildconfig -l app=dev-backend -n node-backend-1'
                    def build = openshift.newBuild(".", "--strategy=source", "--name=dev-backend", "--labels app=dev-backend")
                    build.logs('-f')
                }
                stage('deploy') {
                    openshift.selector("dc", "dev-backend").rollout()
                    def dc = openshift.selector("dc", "dev-backend")
                    dc.rollout().status()
                }
            }
        }
    }
}
