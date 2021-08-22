pipeline {
    agent any
    stages {
        stage ('build pesbuk') {
            steps {
                sh '''
                    sudo docker build -t arizki/pesbuk:$GIT_BRANCH-$BUILD_ID -f pesbuk/Dockerfile.pesbuk .
                    sudo docker login -u arizki -p$DOCKER_TOKEN
                    sudo docker push arizki/pesbuk:$GIT_BRANCH-$BUILD_ID
                '''
            }
        }
        stage ('change manifest file and send') {
            steps {
                sh '''
                    sed -i -e "s/branch/$GIT_BRANCH/" kube/pesbuk-deployment.yml
                    sed -i -e "s/appversion/$BUILD_ID/" pesbuk/pesbuk-deployment.yml
                    tar -czvf manifest.tar.gz kube/*
                '''
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "kube-master",
                            transfers: [sshTransfer(sourceFiles: 'manifest.tar.gz', remoteDirectory: 'jenkins/')],
                            verbose: true
                        )
                    ]
                )
            }
        }
        stage ('deploy to kube cluster') {
            steps {
                sshagent(credentials : ['kube-master-arizki']){
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.jenkins.sdcilsy-alpha.my.id tar -xvzf jenkins/manifest.tar.gz'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.jenkins.sdcilsy-alpha.my.id kubectl apply -f ./kube/namespace/'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.jenkins.sdcilsy-alpha.my.id kubectl apply -f ./kube/pesbuk/'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.jenkins.sdcilsy-alpha.my.id kubectl apply -f ./kube/ingress/'
                }
            }
        }
    }
}