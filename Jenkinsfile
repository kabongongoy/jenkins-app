pipeline {
    agent {
        docker {
            image 'amazon/aws-cli:2.13.0'
            args '-v /root/.aws:/root/.aws --entrypoint=""'  // Clear the ENTRYPOINT
        }
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {
        stage('Generate and Display Token') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'aws', 
                        usernameVariable: 'AWS_ACCESS_KEY_ID', 
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        
                        // Generate a strong random token using openssl
                        def token = sh(returnStdout: true, script: 'openssl rand -base64 32').trim()

                        // Display the token in the Jenkins console
                        echo "Generated token: ${token}"

                        // Save the token to a file temporarily (optional)
                        writeFile(file: 'secret-token.txt', text: token)

                        // Use AWS CLI to store the token in AWS Secrets Manager
                        def secretName = "your-aws-secret-name"
                        def region = "your-aws-region"

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
    }

    post {
        always {
            // Use a generic node to delete the workspace
            node {
                // Clean up the workspace by deleting all files
                deleteDir()
            }
        }
    }
}
