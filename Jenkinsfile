pipeline {
    agent  {
        label 'AGENT-1'
    }
    environment { 
        appVersion = ''
        REGION = "us-east-1"
        ACC_ID = "315069654700"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    // Build
    stages {
        /* stage('Deploy') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
        } */
        stage('Deploy'){
            steps {
                script{
                    withAWS(region: 'us-east-1', credentials: "aws-creds") {
                        sh """
                            echo "Setting up kubeconfig..."
                            aws eks update-kubeconfig --region ${REGION} --name "$PROJECT-${params.deploy_to}"

                            echo "Checking cluster nodes..."
                            kubectl get nodes

                            echo "Preparing Helm deployment..."
                            #sed -i 's/IMAGE_VERSION/${params.appVersion}/g' values-${params.deploy_to}.yaml

                            echo "Deploying new version with Helm..."
                            helm upgrade --install ${COMPONENT} -n ${PROJECT} -f values-${params.deploy_to}.yaml --set deployment.imageVersion=${params.appVersion}.
                        """
                    }
                }
            }
        }
        stage('check status'){
            steps {
                script{
                    withAWS(region: 'us-east-1', credentials: "aws-creds") {
                        def rolloutStatus = sh(
                            script: "kubectl rollout status deployment/${COMPONENT} -n ${PROJECT} || echo FAILED",
                            returnStdout: true
                        ).trim()

                        if (rolloutStatus.contains("successfully rolled out")) {
                            echo "Deployment success"
                        } else {
                            sh """
                                helm rollback ${COMPONENT} -n ${PROJECT}
                                sleep 30
                            """
                            def rollbackStatus = sh(
                                script: "kubectl rollout status deployment/${COMPONENT} -n ${PROJECT} || echo FAILED",
                                returnStdout: true
                            ).trim()
                            if (rollbackStatus.contains("successfully rolled out")) {
                                error "Deployment Failed, Rollback success."
                            }
                            else{
                                error "Deployment Failed, Rollback Failed. Emergency"
                            }
                        }
                    }
                }
            }
        }
    }

    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success { 
            echo 'Hello Success'
        }
        failure { 
            echo 'Hello Failure'
        }
    }
}