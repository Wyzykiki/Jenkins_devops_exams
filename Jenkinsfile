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
                        sh 'docker build -f ./movie-service/Dockerfile -t $DOCKER_ID/movie-service:$DOCKER_TAG ./movie-service'
                    }
                }

                stage('Build Cast Service') {
                    steps {
                        sh 'docker build -f ./cast-service/Dockerfile -t $DOCKER_ID/cast-service:$DOCKER_TAG ./cast-service'
                    }
                }
            }
        }

        stage('Docker Push') {
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }

            steps {
                sh 'docker login -u $DOCKER_ID -p $DOCKER_PASS'
            }

            parallel {
                stage('Push Movie Service') {
                    steps {
                        sh 'docker push $DOCKER_ID/movie-service:$DOCKER_TAG'
                    }
                }

                stage('Push Cast Service') {
                    steps {
                        sh 'docker push $DOCKER_ID/cast-service:$DOCKER_TAG'
                    }
                }
            }

        }

        stage('Deploiement en dev') {
            // environment
            // {
            // KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            // }
            steps {
                sh 'helm upgrade --install jenkins-exam  charts/ --set environment=dev'
            }
        }

        // stage('Deploiement en staging') {
        //     environment
        //     {
        //     KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        //     }
        //         steps {
        //             script {
        //             sh '''
        //             rm -Rf .kube
        //             mkdir .kube
        //             ls
        //             cat $KUBECONFIG > .kube/config
        //             cp fastapi/values.yaml values.yml
        //             cat values.yml
        //             sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        //             helm upgrade --install app fastapi --values=values.yml --namespace staging
        //             '''
        //             }
        //         }

        // }

        // stage('Deploiement en prod') {
        //     environment
        //     {
        //     KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        //     }
        //         steps {
        //         // Create an Approval Button with a timeout of 15minutes.
        //         // this require a manuel validation in order to deploy on production environment
        //                 timeout(time: 15, unit: "MINUTES") {
        //                     input message: 'Do you want to deploy in production ?', ok: 'Yes'
        //                 }

        //             script {
        //             sh '''
        //             rm -Rf .kube
        //             mkdir .kube
        //             ls
        //             cat $KUBECONFIG > .kube/config
        //             cp fastapi/values.yaml values.yml
        //             cat values.yml
        //             sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        //             helm upgrade --install app fastapi --values=values.yml --namespace prod
        //             '''
        //             }
        //         }
        // }
    }
}
