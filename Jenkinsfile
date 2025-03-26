pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        S3_BUCKET = "my-github-backup-bucket-test"
        GIT_REPO = "https://github.com/ishanpathak98/S3-Test.git"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh """
                    rm -rf repo
                    git clone $GIT_REPO repo
                    """
                }
            }
        }

        stage('Verify Repository') {
            steps {
                script {
                    sh """
                    if [ ! -d "repo" ]; then
                      echo "Error: Repository folder not found!"
                      exit 1
                    fi
                    """
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
            echo "✅ GitHub repo successfully cloned and synced to S3!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs!"
        }
    }
}
