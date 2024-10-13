pipeline {
    agent {
        docker {
            image 'amazon/aws-cli:2.13.0'  // Use the AWS CLI image
            args '-v /root/.aws:/root/.aws --entrypoint=""'  // Clear the ENTRYPOINT
        }
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
                        def secretName = "your-aws-secret-name"  // Update with your AWS secret name
                        def region = "your-aws-region"            // Update with your AWS region

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
}
