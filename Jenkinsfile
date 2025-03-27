pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to deploy (main/dev/test/prod)')
    }

    environment {
        AWS_REGION = "us-east-1"
        GIT_REPO = "https://github.com/ishanpathak98/S3-Test.git"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh """
                    rm -rf repo
                    git clone -b ${BRANCH_NAME} ${GIT_REPO} repo
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

        stage('Determine S3 Bucket') {
            steps {
                script {
                    // Map branch name to corresponding S3 bucket dynamically
                    def bucketMap = [
                        'main': 'my-github-backup-bucket-main',
                        'dev': 'my-github-backup-bucket-dev',
                        'test': 'my-github-backup-bucket-test',
                        'prod': 'my-github-backup-bucket-prod'
                    ]
                    
                    if (bucketMap.containsKey(BRANCH_NAME)) {
                        env.S3_BUCKET = bucketMap[BRANCH_NAME]
                    } else {
                        error("❌ Invalid branch name. No S3 bucket mapped for ${BRANCH_NAME}")
                    }
                }
            }
        }

        stage('Sync to S3') {
            steps {
                script {
                    sh """
                    aws s3 sync repo s3://${S3_BUCKET} --delete
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ GitHub repo successfully cloned from branch ${BRANCH_NAME} and synced to S3 bucket: ${S3_BUCKET}!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs!"
        }
    }
}
