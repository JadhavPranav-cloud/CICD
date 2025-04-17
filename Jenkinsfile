pipeline {
    agent any

    environment {
        EC2_2 = "ubuntu@15.207.116.242"
        DEPLOY_PATH = "/var/www/vmedulife"
        DEPLOY_TEMP = "/tmp/deploy-temp"
        BRANCH = "dev"
    }

    stages {
        stage('Deploy to EC2 Instances') {
            steps {
                sshagent (credentials: ['ubuntu']) {
                    script {
                        def servers = [env.EC2_2]
                        for (server in servers) {
                            sh """
                                echo "Deploying to ${server}..."

                                ssh -o StrictHostKeyChecking=no ${server} 'sudo mkdir -p ${DEPLOY_PATH} && mkdir -p ${DEPLOY_TEMP}'

                                scp -o StrictHostKeyChecking=no -r website/* ${server}:${DEPLOY_TEMP}/

                                ssh -o StrictHostKeyChecking=no ${server} 'sudo rm -rf ${DEPLOY_PATH}/*'
                                ssh -o StrictHostKeyChecking=no ${server} 'sudo mv ${DEPLOY_TEMP}/* ${DEPLOY_PATH}/'
                                ssh -o StrictHostKeyChecking=no ${server} 'sudo rm -rf ${DEPLOY_TEMP}'
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
            mail to: 'pranav.jadhav@corp.vmedulife.com',
                 subject: "✅ SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: "The deployment completed successfully.\n\nConsole output:\n${env.BUILD_URL}console"
        }
        failure {
            mail to: 'pranav.jadhav@corp.vmedulife.com',
                 subject: "❌ FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: "The deployment failed. Please check the logs.\n\nConsole output:\n${env.BUILD_URL}console"
        }
    }
}
