pipeline {
    agent any

    environment {
        EC2_2 = "ubuntu@${EC2_2_IP}"
        DEPLOY_PATH = "/var/www/vmedulife"
        DEPLOY_TEMP = "/tmp/deploy-temp"
        BRANCH = "dev"
        ZOHO_WEBHOOK_URL = "https://cliq.zoho.com/company/711832931/integrations/bots/b-2760934000010255033/edit/9"
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

                                ssh -o StrictHostKeyChecking=no ${server} 'sudo mkdir -p ${env.DEPLOY_PATH} && mkdir -p ${env.DEPLOY_TEMP}'
                                scp -o StrictHostKeyChecking=no -r website/* ${server}:${env.DEPLOY_TEMP}/
                                ssh -o StrictHostKeyChecking=no ${server} 'sudo rm -rf ${env.DEPLOY_PATH}/*'
                                ssh -o StrictHostKeyChecking=no ${server} 'sudo mv ${env.DEPLOY_TEMP}/* ${env.DEPLOY_PATH}/'
                                ssh -o StrictHostKeyChecking=no ${server} 'sudo rm -rf ${env.DEPLOY_TEMP}'
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
                
Build Number: #${env.BUILD_NUMBER}"""

            def payload = """{
                "text": "✅ Successfully Deployed!\\nBuild Number: #${env.BUILD_NUMBER}\\n[View Console Output](${env.BUILD_URL}console)"
            }"""

            sh """
                curl -X POST -H 'Content-Type: application/json' -d '${payload}' ${env.ZOHO_WEBHOOK_URL}
            """
        }
    }
    failure {
        script {
            mail to: 'pranav.jadhav@corp.vmedulife.com',
                 
Build Number: #${env.BUILD_NUMBER}"""

            def payload = """{
                "text": "❌ Deployment Failed!\\nBuild Number: #${env.BUILD_NUMBER}\\n[View Console Output](${env.BUILD_URL}console)"
            }"""
            sh """
                curl -X POST -H 'Content-Type: application/json' -d '${payload}' ${env.ZOHO_WEBHOOK_URL}
            """
        }
    }
}

}
