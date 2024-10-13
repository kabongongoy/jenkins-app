pipeline {
    agent none  // Define that we will use specific agents for stages

    stages {
        stage('Generate Token') {
            agent any  // This runs on any available Jenkins agent
            steps {
                script {
                    // Generate a strong random token using openssl
                    def token = sh(returnStdout: true, script: 'openssl rand -base64 32').trim()

                    // Display the token in the Jenkins console
                    echo "Generated token: ${token}"

                    // Save the token to a file temporarily
                    writeFile(file: 'secret-token.txt', text: token)
                }
            }
        }

        stage('Store in AWS Secrets Manager') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.13.0'  // Use the AWS CLI Docker image
                    args '-v /root/.aws:/root/.aws' // Mount AWS credentials if needed
                }
            }
            steps {
                script {
                    // Use AWS CLI to store the token in AWS Secrets Manager
                    def secretName = "your-aws-secret-name"  // Update with your AWS secret name
                    def region = "us-east-1"            // Update with your AWS region

                    // Store the token in AWS Secrets Manager using AWS CLI
                    sh """
                        aws secretsmanager put-secret-value \
                        --secret-id ${secretName} \
                        --secret-string file://secret-token.txt \
                        --region ${region}
                    """
                }
            }
        }
    }

    post {
        always {
            deleteDir()  // Clean up the workspace
        }
    }
}
