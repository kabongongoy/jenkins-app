pipeline {
    agent any

    stages {
        stage('Generate Secret Token') {
            steps {
                script {
                    // Generate a random secret token
                    def secretToken = sh(script: 'openssl rand -base64 32', returnStdout: true).trim()
                    echo "Generated Secret Token: ${secretToken}"

                    // Save the token to a file for use in AWS Secrets Manager
                    writeFile file: 'secret-token.txt', text: secretToken
                }
            }
        }

        stage('List S3 Buckets') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.13.0'  // Use the AWS CLI Docker image
                    args '--entrypoint="" -u root'  // Run as root user for permission
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

                        // List S3 buckets
                        sh """
                            echo "Listing S3 Buckets..."
                            aws s3 ls --region us-east-1 --debug
                        """
                    }
                }
            }
        }

        stage('Store in AWS Secrets Manager') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.13.0'  // Use the AWS CLI Docker image
                    args '--entrypoint="" -u root'  // Run as root user for permission
                }
            }
            steps {
                // Bind the AWS credentials using Jenkins credentials
                withCredentials([usernamePassword(credentialsId: 'aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    script {
                        // Use AWS CLI to store the token in AWS Secrets Manager
                        def secretName = "your-aws-secret-name"  // Update with your AWS secret name
                        def region = "us-east-1"                   // Update with your AWS region

                        // Try to create the secret
                        try {
                            sh """
                                aws secretsmanager create-secret \
                                --name ${secretName} \
                                --secret-string file://secret-token.txt \
                                --region ${region}
                            """
                        } catch (Exception e) {
                            echo "Secret already exists, updating..."
                        }

                        // Update the secret
                        def response = sh(script: """
                            aws secretsmanager put-secret-value \
                            --secret-id ${secretName} \
                            --secret-string file://secret-token.txt \
                            --region ${region} --debug
                        """, returnStdout: true).trim()

                        echo "Response from Secrets Manager: ${response}"
                    }
                }
            }
        }
    }
}
