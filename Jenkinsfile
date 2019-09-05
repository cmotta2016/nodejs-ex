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
        stage('Code Quality'){
            
            def final scannerHome = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
            
            def version = sh (script: 'PACKAGE_VERSION=$(cat package.json | grep version | head -1 | awk -F: \'{ print $2 }\' | sed \'s/[\\",]//g\' | tr -d \'[[:space:]]\') && echo $PACKAGE_VERSION', returnStdout: true).trim()
            def git_branch = sh (script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
            git_branch = (git_branch == 'master' || git_branch == 'HEAD')?'': git_branch
            def git_repoURL = sh (script: 'git config --get remote.origin.url', returnStdout: true).trim()
            
            sonarProperties = readFile 'sonar-project.properties'
            sonarProperties = sonarProperties?.replaceAll('@version', version)?.replaceAll('@git_branch', git_branch)?.replaceAll('@git_repoURL', git_repoURL)?.replaceAll('@job_URL', "${JOB_URL}")
            
            // Fazer o print 
            println sonarProperties
            println(scannerHome)
            writeFile encoding: 'UTF-8', file: 'sonar-project.properties', text: sonarProperties
            sh 'ls -las /'
            sh 'java -version'
            withSonarQubeEnv('SonarQube'){
                sh "${scannerHome}/bin/sonar-scanner"
            }
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
