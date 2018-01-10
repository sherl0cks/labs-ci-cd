node() {
    stage('Initialization') {
        env.OCP_API_SERVER = "${env.OPENSHIFT_API_URL}"
        env.OCP_TOKEN = readFile('/var/run/secrets/kubernetes.io/serviceaccount/token').trim()
    }
}
timeout(time: 60, unit: 'SECONDS') {
    def userInput = input(
            id: 'userInput', message: 'Which PR # do you want to test?', parameters: [
            [$class: 'TextParameterDefinition', description: 'git ref', name: 'ref']
    ])
    echo ("Env: "+userInput)
}

node(){

    stage('Checkout SCM'){
        sh 'rm -rf labs-ci-cd && git clone https://github.com/rht-labs/labs-ci-cd'
    }

    stage('Merge SCM'){
        sh "git fetch origin pull/${userInput}/head:pr && git merge pr -ff"
    }

    stage('Apply Inventory'){
        def sha = sh(returnStdout: true, script: 'cd labs-ci-cd && git rev-parse HEAD')
        echo sha
    }

    stage('Verify Results'){
        parallel(
                'CI Builds': {
                    String[] builds = ['mvn-build-pod','npm-build-pod','zap-build-pod']

                    for (String build : builds ){
                        openshiftVerifyBuild (
                                apiURL: "${env.OCP_API_SERVER}",
                                authToken: "${env.OCP_TOKEN}",
                                buildConfig: build,
                                namespace: "labs-ci-cd",
                                waitTime: '10',
                                waitUnit: 'min'
                        )
                    }
                },
                'CI Deploys': {
                    String[] ciDeployments = ['jenkins','nexus','sonarqube']

                    for (String deploy : ciDeployments ){
                        openshiftVerifyDeployment (
                                apiURL: "${env.OCP_API_SERVER}",
                                authToken: "${env.OCP_TOKEN}",
                                depCfg: deploy,
                                namespace: "labs-ci-cd",
                                verifyReplicaCount: true,
                                waitTime: '10',
                                waitUnit: 'min'
                        )
                    }
                },
                'App Deploys': {
                    String[] appDeployments = ['java-app']

                    for (String app : appDeployments ){
                        openshiftVerifyDeployment (
                                apiURL: "${env.OCP_API_SERVER}",
                                authToken: "${env.OCP_TOKEN}",
                                depCfg: app,
                                namespace: "labs-dev",
                                verifyReplicaCount: true,
                                waitTime: '10',
                                waitUnit: 'min'
                        )
                    }
                },failFast: true
        )
    }

    stage('Update PR'){
    }

    // stage('Verify CI/CD Deployments') {


    // }

}
