pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'stg', 'prod'],
            description: 'Select target environment'
        )
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Workspace Select') {
            steps {
                sh '''
                    terraform workspace select ${ENVIRONMENT} || terraform workspace new ${ENVIRONMENT}
                '''
            }
        }

        stage('Terraform Plan') {
            steps {
                sh '''
                    if [ "${ENVIRONMENT}" = "dev" ]; then
                        terraform plan -var-file=dev -out=tfplan
                    elif [ "${ENVIRONMENT}" = "stg" ]; then
                        terraform plan -var-file=stg.tfvars -out=tfplan
                    elif [ "${ENVIRONMENT}" = "prod" ]; then
                        terraform plan -var-file=Prod -out=tfplan
                    fi
                '''
            }
        }

        stage('Manual Approval') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: "Proceed with changes for ${params.ENVIRONMENT}?", ok: "Apply"
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                sh 'terraform apply -input=false tfplan'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }

        success {
            emailext(
                subject: "SUCCESS: Job '${env.JOB_NAME}' [Build #${env.BUILD_NUMBER}]",
                body: """Hello,

Pipeline executed successfully for environment: ${params.ENVIRONMENT}

Details:
- Job: ${env.JOB_NAME}
- Build: ${env.BUILD_NUMBER}
- URL: ${env.BUILD_URL}
""",
                to: 'ahmedfaroukcvx@gmail.com'
            )
        }

        failure {
            emailext(
                subject: "FAILED: Job '${env.JOB_NAME}' [Build #${env.BUILD_NUMBER}]",
                body: """Hello,

Pipeline failed for environment: ${params.ENVIRONMENT}

Details:
- Job: ${env.JOB_NAME}
- Build: ${env.BUILD_NUMBER}
- URL: ${env.BUILD_URL}
""",
                to: 'ahmedfaroukcvx@gmail.com'
            )
        }
    }
}
