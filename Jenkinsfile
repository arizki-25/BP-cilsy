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
        stage ('build wordpress') {
            steps {
                sh '''
                    sudo docker build -t arizki/wordpress:$GIT_BRANCH-$BUILD_ID -f wordpress/Dockerfile .
                    sudo docker login -u arizki -p$DOCKER_TOKEN
                    sudo docker push arizki/wordpress:$GIT_BRANCH-$BUILD_ID
                '''
            }
        }
        stage ('change manifest file and send') {
            stage ('for production') {
                when {
                    branch 'main'
                }
                steps {
                    sh '''
                        sed -i -e "s/branch/$GIT_BRANCH/" kube/production/pesbuk/pesbuk-deployment.yml
                        sed -i -e "s/appversion/$BUILD_ID/" kube/production/pesbuk/pesbuk-deployment.yml
                        sed -i -e "s/branch/$GIT_BRANCH/" kube/production/landingpage/landing-page.yml
                        sed -i -e "s/appversion/$BUILD_ID/" kube/production/landingpage/landing-page.yml
                        sed -i -e "s/branch/$GIT_BRANCH/" kube//production/wordpress/wordpress-deployment.yml
                        sed -i -e "s/appversion/$BUILD_ID/" kube/production/wordpress/wordpress-deployment.yml
                        tar -czvf manifest-production.tar.gz kube/production/ kube/ingress/
                    '''
                    sshPublisher(
                        continueOnError: false, 
                        failOnError: true,
                        publishers: [
                            sshPublisherDesc(
                                configName: "kube-master",
                                transfers: [sshTransfer(sourceFiles: 'manifest-production.tar.gz', remoteDirectory: 'jenkins/')],
                                verbose: true
                            )
                        ]
                    )
                }
            }
            stage ('for staging') {
                when {
                    branch 'staging'
                }
                steps {
                    sh '''
                        sed -i -e "s/branch/$GIT_BRANCH/" kube/staging/pesbuk/pesbuk-deployment.yml
                        sed -i -e "s/appversion/$BUILD_ID/" kube/staging/pesbuk/pesbuk-deployment.yml
                        sed -i -e "s/branch/$GIT_BRANCH/" kube/staging/landingpage/landing-page.yml
                        sed -i -e "s/appversion/$BUILD_ID/" kube/staging/landingpage/landing-page.yml
                        sed -i -e "s/branch/$GIT_BRANCH/" kube//staging/wordpress/wordpress-deployment.yml
                        sed -i -e "s/appversion/$BUILD_ID/" kube/staging/wordpress/wordpress-deployment.yml
                        tar -czvf manifest-staging.tar.gz kube/staging/ kube/ingress/
                    '''
                    sshPublisher(
                        continueOnError: false, 
                        failOnError: true,
                        publishers: [
                            sshPublisherDesc(
                                configName: "k8s-master",
                                transfers: [sshTransfer(sourceFiles: 'manifest-staging.tar.gz', remoteDirectory: 'jenkins/')],
                                verbose: true
                            )
                        ]
                    )
                }
            }
        }
        stage ('deploy to kube cluster') {
            stage ('for production') {
                when {
                    branch 'main'
                }
                steps {
                    sshagent(credentials : ['kube-master-arizki']){
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id tar -xvzf jenkins/manifest.tar.gz'
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id kubectl apply -f kube/production/namespace'
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id kubectl apply -f kube/production/landingpage'
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id kubectl apply -f kube/production/pesbuk'
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id kubectl apply -f kube/production/wordpress'
                    }
                }
            }
           stage ('for staging') {
                when {
                    branch 'staging'
                }
                steps {
                    sshagent(credentials : ['kube-master-arizki']){
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id tar -xvzf jenkins/manifest.tar.gz'
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id kubectl apply -f kube/staging/namespace'
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id kubectl apply -f kube/staging/landingpage'
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id kubectl apply -f kube/staging/pesbuk'
                        sh 'ssh -o StrictHostKeyChecking=no ubuntu@api.k8s.sdcilsy-alpha.my.id kubectl apply -f kube/staging/wordpress'
                    }
                }
            }
        }
    } 
}
