pipeline {
    agent any
    stages {
        stage ('build pesbuk') {
            steps {
                sh '''
                    sudo docker build -t arizki/pesbuk:$GIT_BRANCH-$BUILD_ID -f pesbuk/Dockerfile .
                    sudo docker login -u arizki -p$DOCKER_TOKEN
                    sudo docker push arizki/pesbuk:$GIT_BRANCH-$BUILD_ID
                '''
            }
        }
        stage ('build landing-page') {
            steps {
                sh '''
                    sudo docker build -t arizki/landingpage:$GIT_BRANCH-$BUILD_ID -f landing-page/Dockerfile .
                    sudo docker login -u arizki -p$DOCKER_TOKEN
                    sudo docker push arizki/landingpage:$GIT_BRANCH-$BUILD_ID
                '''
            }
        }
        stage ('change manifest file and send') {
            steps {
                sh '''
                    sed -i -e "s/branch/$GIT_BRANCH/" kube/pesbuk/pesbuk-deployment.yml
                    sed -i -e "s/appversion/$BUILD_ID/" kube/pesbuk/pesbuk-deployment.yml
                    sed -i -e "s/branch/$GIT_BRANCH/" kube/landingpage/landing-page.yml
                    sed -i -e "s/appversion/$BUILD_ID/" kube/landingpage/landing-page.yml
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
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id tar -xvzf jenkins/manifest.tar.gz'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id kubectl apply -f kube/namespace'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id kubectl apply -f kube'
                }
            }
        }
    }
}