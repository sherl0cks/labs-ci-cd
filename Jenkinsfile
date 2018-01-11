node() {
    try {
        stage('Initialization') {
            env.OCP_API_SERVER = "${env.OPENSHIFT_API_URL}"
            env.OCP_TOKEN = readFile('/var/run/secrets/kubernetes.io/serviceaccount/token').trim()
        }

        stage('Checkout SCM') {
            sh 'rm -rf labs-ci-cd && git clone https://github.com/rht-labs/labs-ci-cd'
        }

        stage('Merge SCM') {

            sh 'git config --global user.email "robot@example.com" && git config --global user.name "A Robot"'

            timeout(time: 60, unit: 'SECONDS') {
                def userInput = input(
                        id: 'userInput', message: 'Which PR # do you want to test?', parameters: [
                        [$class: 'StringParameterDefinition', description: 'PR #', name: 'pr'],
                        [$class: 'StringParameterDefinition', description: 'github token', name: 'token']
                ])
                env.PR_ID = userInput['pr']
                env.PR_GITHUB_TOKEN = userInput['token']

                echo ("${env.PR_ID} ${env.PR_GITHUB_TOKEN}")
            }

            // set the vars
            env.COMMIT_SHA = sh(returnStdout: true, script: "cd labs-ci-cd && git fetch origin pull/${env.PR_ID}/head:pr && git checkout pr && git rev-parse HEAD")
            env.USER_PASS = "sherl0cks:${env.PR_GITHUB_TOKEN}"
            env.PR_STATUS_URI = "https://api.github.com/repos/rht-labs/labs-ci-cd/statuses/${env.COMMIT_SHA}"

            // do a merge
            // this will also
            sh(returnStdout: true, script: "cd labs-ci-cd && git checkout master && git fetch origin pull/${env.PR_ID}/head:pr && git merge pr --ff")



            if (env.PR_GITHUB_TOKEN == null || env.PR_GITHUB_TOKEN == "") {
                // skip github statusing
            } else {
                def json = '''\
                {
                    "state": "pending",
                    "description": "job is running...",
                    "context": "Jenkins"
                }'''

                sh "curl -u ${env.USER_PASS} -d '${json}' -H 'Content-Type: application/json' -X POST ${env.PR_STATUS_URI}"

            }

        }

        stage('Apply Inventory') {
            node('jenkins-slave-ansible') {
                sh 'git config --global user.email "robot@example.com" && git config --global user.name "A Robot"'
                sh "git clone https://github.com/rht-labs/labs-ci-cd && cd labs-ci-cd && git fetch origin pull/${env.PR_ID}/head:pr && git merge pr --ff"
                sh(returnStdout: true, script: "cd labs-ci-cd && ansible-galaxy install -r requirements.yml --roles-path=roles")
                sh(returnStdout: true, script: "cd labs-ci-cd && ansible-playbook roles/casl-ansible/playbooks/openshift-cluster-seed.yml -i inventory/")
            }
        }

        stage('Verify Results') {
            parallel(
                    'CI Builds': {
                        String[] builds = ['mvn-build-pod', 'npm-build-pod', 'zap-build-pod']

                        for (String build : builds) {
                            openshiftVerifyBuild(
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
                        String[] ciDeployments = ['jenkins', 'nexus', 'sonarqube']

                        for (String deploy : ciDeployments) {
                            openshiftVerifyDeployment(
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

                        for (String app : appDeployments) {
                            openshiftVerifyDeployment(
                                    apiURL: "${env.OCP_API_SERVER}",
                                    authToken: "${env.OCP_TOKEN}",
                                    depCfg: app,
                                    namespace: "labs-dev",
                                    verifyReplicaCount: true,
                                    waitTime: '10',
                                    waitUnit: 'min'
                            )
                        }
                    }, failFast: true
            )

            def json = '''\
            {
                "state": "success",
                "description": "the job passed!",
                "context": "Jenkins"
            }'''

            sh "curl -u ${env.USER_PASS} -d '${json}' -H 'Content-Type: application/json' -X POST ${env.PR_STATUS_URI}"
        }

    }
    catch (e){

        if (env.USER_PASS == null || env.USER_PASS == ""){
            error("The PR # you entered was bad and the job failed. Please double check the github issue ID")
        } else {
            def json = '''\
        {
            "state": "failure",
            "description": "the job failed",
            "context": "Jenkins"
        }'''

            sh "curl -u ${env.USER_PASS} -d '${json}' -H 'Content-Type: application/json' -X POST ${env.PR_STATUS_URI}"
        }

        throw e

    }
}