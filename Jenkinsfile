pipeline {
    agent any

    environment {
        EC2_2 = "ubuntu@${EC2_2_IP}"
        DEPLOY_PATH = "/var/www/vmedulife"
        DEPLOY_TEMP = "/tmp/deploy-temp"
        BRANCH = "dev"
        ZOHO_WEBHOOK_URL = "https://cliq.zoho.com/company/711832931/api/v2/bots/jenkinsbot/incoming"
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
            script {
                mail to: 'pranav.jadhav@corp.vmedulife.com',
                     subject: "✅ Jenkins Build #${env.BUILD_NUMBER} Succeeded",
                     body: "Deployment was successful."

                def payload = '''{
                    "text": "✅ Successfully Deployed!\\nBuild Number: #${env.BUILD_NUMBER}"
                }'''
                sh """
                    curl -X POST -H 'Content-Type: application/json' -d '${payload}' ${env.ZOHO_WEBHOOK_URL}
                """
            }
        }

        failure {
            script {
                mail to: 'pranav.jadhav@corp.vmedulife.com',
                     subject: "❌ Jenkins Build #${env.BUILD_NUMBER} Failed",
                     body: "Deployment failed."

                def payload = '''{
                    "text": "❌ Deployment Failed!\\nBuild Number: #${env.BUILD_NUMBER}"
                }'''
                sh """
                    curl -X POST -H 'Content-Type: application/json' -d '${payload}' ${env.ZOHO_WEBHOOK_URL}
                """
            }
        }
    }
}
