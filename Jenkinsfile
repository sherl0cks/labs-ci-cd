node() {
    stage('Initialization') {
        env.OCP_API_SERVER = "${env.OPENSHIFT_API_URL}"
        env.OCP_TOKEN = readFile('/var/run/secrets/kubernetes.io/serviceaccount/token').trim()
    }
}


node(){

    stage('Checkout SCM'){
        sh 'rm -rf labs-ci-cd && git clone https://github.com/rht-labs/labs-ci-cd'
    }

    stage('Merge SCM'){

        sh 'git config --global user.email "robot@example.com" && git config --global user.name "A Robot"'

        // timeout(time: 60, unit: 'SECONDS') {
        //     def userInput = input(
        //      id: 'userInput', message: 'Which PR # do you want to test?', parameters: [
        //      [$class: 'StringParameterDefinition', description: 'PR #', name: 'pr'],
        //      [$class: 'StringParameterDefinition', description: 'github token', name: 'token']
        //     ])
        //     env.PR_ID = userInput['pr']
        //     env.PR_GITHUB_TOKEN = userInput['token']

        //     echo ("${env.PR_ID} ${env.PR_GITHUB_TOKEN}")
        // }

        env.PR_GITHUB_TOKEN = 'f87853c93501738745963914d9dd1414b5d5d029'
        env.PR_ID = 72
        env.COMMIT_SHA = sh(returnStdout: true, script: 'cd labs-ci-cd && git rev-parse HEAD')

        if (env.PR_GITHUB_TOKEN == null || env.PR_GITHUB_TOKEN == "") {
            // skip github statusing
        } else {
            def json = '''\
            {
                "state": "pending",
                "description": "job is running...",
                "context": "Jenkins"
            }'''

            echo(json)
            def uri = "https://api.github.com/repos/rht-labs/labs-ci-cd/statuses/ab9aa4877324583ad88cc41d41f66e6be04ac06e"
            def userPass = "sherl0cks:${env.PR_GITHUB_TOKEN}"
            sh "curl -u ${userPass} -d '${json}' -H 'Content-Type: application/json' -X POST ${uri}"

        }

        sh(returnStdout: true, script: "cd labs-ci-cd && git fetch origin pull/${env.PR_ID}/head:pr && git merge pr --ff")

    }

    stage('Apply Inventory'){

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
