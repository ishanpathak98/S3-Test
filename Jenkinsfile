pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        S3_BUCKET = "my-github-backup-bucket-test"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh 'rm -rf repo && git clone https://github.com/ishanpathak98/S3-Test.git'
                }
            }
        }

        stage('Sync to S3') {
            steps {
                script {
                    sh """
                    aws s3 sync repo s3://$S3_BUCKET --delete
                    """
                }
            }
        }
    }

    post {
        success {
            echo "GitHub repo successfully cloned to S3!"
        }
        failure {
            echo "Pipeline failed. Check logs!"
        }
    }
}
