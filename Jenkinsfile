pipeline {
    agent any  // Use any available agent for this job

    stages {
        stage('Generate and Display Token') {
            steps {
                script {
                    // Generate a strong random token using openssl
                    def token = sh(returnStdout: true, script: 'openssl rand -base64 32').trim()

                    // Display the token in the Jenkins console
                    echo "Generated token: ${token}"
                }
            }
        }
    }
}
