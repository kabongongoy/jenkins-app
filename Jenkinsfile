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
                    args '--entrypoint=""'           // Override the default entrypoint
                    args '-u root'                   // Run as root user for permission
                }
            }
            steps {
                // Bind the AWS credentials using Jenkins credentials
                withCredentials([usernamePassword(credentialsId: 'aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    script {
                        // Create AWS credentials file for the Docker container
                        sh """
                            mkdir -p /root/.aws
                            echo "[default]" > /root/.aws/config
                            echo "aws_access_key_id=${AWS_ACCESS_KEY_ID}" >> /root/.aws/config
                            echo "aws_secret_access_key=${AWS_SECRET_ACCESS_KEY}" >> /root/.aws/config
                        """

                        // Use AWS CLI to store the token in AWS Secrets Manager
                        def secretName = "your-aws-secret-name"  // Update with your AWS secret name
                        def region = "us-east-1"            // Update with your AWS region

                        // Store the token in AWS Secrets Manager using AWS CLI
                        sh """
                            aws secretsmanager put-secret-value \
                            --secret-id ${secretName} \
                            --secret-string file://secret-token.txt \
                            --region ${region} --debug
                        """
                    }
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
