node() {
    try {
        stage ("Get Latest Code") {
            checkout scm
            sh 'git rev-parse HEAD > .git/commit-id'
        }
	stage ("teste") {
	    sh 'echo hello'
        }
        stage ("Install Application Dependencies") {
            sh 'sudo pip install --upgrade ansible==${ANSIBLE_VERSION} molecule==${MOLECULE_VERSION} docker'
        }
        stage ("Executing Molecule lint") {
            sh 'molecule lint'
        }
        stage ("Executing Molecule create") {
            sh 'molecule create'
        }
        stage ("Executing Molecule converge") {
            sh 'molecule converge'
        }
        stage ("Executing Molecule idempotence") {
            sh 'molecule idempotence'
        }
        stage ("Executing Molecule verify") {
            sh 'molecule verify'
        }
        stage('Tag git'){
            def commit_id = readFile('.git/commit-id').trim()
            withEnv(["COMMIT_ID=${commit_id}"]){
                sh '''#!/bin/bash
                if [[ $(git tag | grep "^${$COMMIT_ID}$" | wc -l) -eq 1 ]]
                    then    echo "Tag already exists"
                    else    echo "Tag will be created"
                            git config user.name "jenkins"
                            git config user.email "jenkins@localhost"
                            git tag -a $COMMIT_ID -m "Added tagging"
                            git push --tags
                fi
                '''
            }
        }
        stage('Start Staging Job') {
            def commit_id = readFile('.git/commit-id').trim()
            withEnv(["COMMIT_ID=${commit_id}"]){
                build job: 'ansible-access-2-staging', wait: false, parameters: [string(name: 'COMMIT_ID', value: "${COMMIT_ID}") ]
            }
        }
    } catch(all) {
        currentBuild.result = "FAILURE"
        throw err
    }
}

