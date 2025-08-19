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
        stage('Deploy') {
            steps {
                script{
                    withAWS(region: 'us-east-1', credentials: "aws-creds") {
                        try {
                    sh """
                        echo "Setting up kubeconfig..."
                        aws eks update-kubeconfig --region ${REGION} --name "$PROJECT-${params.deploy_to}"

                        echo "Checking cluster nodes..."
                        kubectl get nodes

                        echo "Preparing Helm deployment..."
                        cd helm
                        sed -i 's/IMAGE_VERSION/${params.appVersion}/g' values-${params.deploy_to}.yaml

                        echo "Deploying new version with Helm..."
                        helm upgrade --install ${COMPONENT} -n ${PROJECT} -f values-${params.deploy_to}.yaml .
                    """
                } catch (err) {
                    echo "Helm upgrade failed. Attempting rollback..."

                    // Rollback to previous version
                    sh """
                        helm rollback ${COMPONENT} -n ${PROJECT}
                        sleep 30  # Wait for pods to stabilize
                    """

                    // Check rollout status after rollback
                    def rolloutStatus = sh(
                        script: "kubectl rollout status deployment/${COMPONENT} -n ${PROJECT} || echo FAILED",
                        returnStdout: true
                    ).trim()

                    if (rolloutStatus.contains("successfully rolled out")) {
                        echo "Rollback succeeded. But pipeline will fail to notify issue with new release."
                        error("New deployment failed, rollback successful. Please check the new version.")
                    } else {
                        error("Rollback also failed. Previous version is not running.")
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