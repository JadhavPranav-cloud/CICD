pipeline {
    agent any

    environment {
        EC2_2 = "ubuntu@${EC2_2_IP}"
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
                 body: """✅ Successfully deployed!
Build Number: #${env.BUILD_NUMBER}"""
        }
        failure {
            mail to: 'pranav.jadhav@corp.vmedulife.com',
                 subject: "❌ FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: """❌ Deployment failed.
Build Number: #${env.BUILD_NUMBER}"""
        }
    }
}
