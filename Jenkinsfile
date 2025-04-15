pipeline {
    agent any

    environment {
        EC2_1 = "ubuntu@15.206.81.75"
        EC2_2 = "ubuntu@15.207.247.128"
        DEPLOY_PATH = "/var/www/vmedulife"
        BRANCH = "dev"
    }

    stages {
        stage('Deploy to EC2 Instances') {
            steps {
                sshagent (credentials: ['ubuntu']) {
                    script {
                        def servers = [env.EC2_1, env.EC2_2]
                        for (server in servers) {
                            sh """
                                echo "Deploying to ${server}..."

                                ssh -o StrictHostKeyChecking=no ${server} 'sudo mkdir -p ${DEPLOY_PATH} && mkdir -p /tmp/deploy-temp'

                                scp -o StrictHostKeyChecking=no -r website/* ${server}:/tmp/deploy-temp/

                                ssh -o StrictHostKeyChecking=no ${server} 'sudo rm -rf ${DEPLOY_PATH}/*'
                                ssh -o StrictHostKeyChecking=no ${server} 'sudo mv /tmp/deploy-temp/* ${DEPLOY_PATH}/'
                                ssh -o StrictHostKeyChecking=no ${server} 'sudo rm -rf /tmp/deploy-temp'
                                ssh -o StrictHostKeyChecking=no ${server} 'sudo systemctl restart nginx'
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
