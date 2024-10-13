pipeline {
    agent any  // Use any available agent for this job

    environment {
        // AWS credentials configured in Jenkins (replace 'aws-access-key-id' with your credentials ID)
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {
        stage('Generate Secret') {
            agent {
                node {
                    label 'any'
                }
            }
            steps {
                script {
                    // Generate a strong random token using openssl
                    def token = sh(returnStdout: true, script: 'openssl rand -base64 32').trim()

                    // Save the token to a file temporarily
                    writeFile(file: 'secret-token.txt', text: token)
                    echo "Secret token generated: ${token}"
                }
            }
        }

        stage('Store Secret in AWS Secrets Manager') {
            agent {
                node {
                    label 'any'
                }
            }
            steps {
                script {
                    // Define your secret name
                    def secretName = "your-aws-secret-name"  // Update with your secret name
                    def region = "your-aws-region"            // Update with your AWS region

                    // Use AWS CLI to store the secret
                    sh """
                        aws secretsmanager put-secret-value \
                        --secret-id ${secretName} \
                        --secret-string file://secret-token.txt \
                        --region ${region}
                    """

                    echo "Secret stored in AWS Secrets Manager with name: ${secretName}"
                }
            }
        }
    }

    post {
        always {
            node {  // Ensure deleteDir is inside a node block
                deleteDir()  // Clean up all files created during the build
            }
        }
    }
}
