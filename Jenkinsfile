pipeline {
    environment { // Declaration of environment variables
        DOCKER_ID = "wyzykiki"
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
    }

    agent any // Jenkins will be able to select all available agents

    stages {
        stage('Docker Build') { // docker build image stage
            parallel {
                stage('Build Movie Service') {
                    steps {
                        sh 'docker build -f ./movie-service/Dockerfile -t $DOCKER_ID/movie-service:$DOCKER_TAG -t $DOCKER_ID/movie-service:latest ./movie-service'
                    }
                }

                stage('Build Cast Service') {
                    steps {
                        sh 'docker build -f ./cast-service/Dockerfile -t $DOCKER_ID/cast-service:$DOCKER_TAG -t $DOCKER_ID/cast-service:latest ./cast-service'
                    }
                }
            }
        }

        stage('Docker Auth') {
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }

            steps {
                sh 'docker login -u $DOCKER_ID -p $DOCKER_PASS'
            }
        }

        stage('Docker Push') {
            parallel {
                stage('Push Movie Service') {
                    steps {
                        sh 'docker push $DOCKER_ID/movie-service:$DOCKER_TAG'
                        sh 'docker push $DOCKER_ID/movie-service:latest'
                    }
                }

                stage('Push Cast Service') {
                    steps {
                        sh 'docker push $DOCKER_ID/cast-service:$DOCKER_TAG'
                        sh 'docker push $DOCKER_ID/cast-service:latest'
                    }
                }
            }
        }

        stage('Deploiement en dev') {
            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }

            steps {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install jenkins-exam charts/ --namespace dev
                '''
            }
        }

        stage('Deploiement en QA') {
            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }

            steps {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install jenkins-exam charts/ --namespace qa
                '''
            }
        }

        stage('Deploiement en staging') {
            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }

            steps {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install jenkins-exam charts/ --namespace staging
                '''
            }
        }

        stage('Deploiement en production') {
            when {
                equals expected: 'origin/master', actual: "${GIT_BRANCH}"
            }

            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }

            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }

                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install jenkins-exam charts/ --namespace prod
                '''
            }
        }
    }
}
