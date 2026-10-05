pipeline {
    agent {
        kubernetes {
            yamlFile 'agent.yaml'
        }
    }

    environment {
        IMAGE_NAME = "panavalong/lab3"
        IMAGE_TAG  = "pablo-anavalon"
        K8S_NAMESPACE = "ns-pablo-anavalon"
    }

    stages {

        stage('install') {
            steps {
                container('node') {
                    sh '''
                        corepack enable
                        pnpm install --frozen-lockfile
                    '''
                }
            }
        }

        stage('test') {
            steps {
                container('node') {
                    sh '''
                        pnpm test
                    '''
                }
            }
        }

        stage('build') {
            steps {
                container('docker') {
                    sh '''
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('push') {
            steps {
                container('docker') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASSWORD'
                        )
                    ]) {
                        sh '''
                            echo "${DOCKER_PASSWORD}" | docker login \
                              -u "${DOCKER_USER}" \
                              --password-stdin

                            docker push ${IMAGE_NAME}:${IMAGE_TAG}

                            docker logout
                        '''
                    }
                }
            }
        }

        stage('deploy') {
            steps {
                container('kubectl') {
                    sh '''
                        kubectl apply -f entrega.yaml

                        kubectl rollout status \
                          deployment/deployment-pablo-anavalon \
                          -n ${K8S_NAMESPACE} \
                          --timeout=120s
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline ejecutado correctamente'
        }

        failure {
            echo 'El pipeline falló. Revisar el stage y los logs.'
        }

        always {
            echo 'Pipeline finalizado.'
        }
    }
}