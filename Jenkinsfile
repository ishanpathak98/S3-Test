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
                    git clone -b ${params.BRANCH_NAME} ${env.GIT_REPO} repo
                    """
                }
            }
        }

        stage('Verify Repository') {
            steps {
                script {
                    if (!fileExists("repo")) {
                        error "❌ Error: Repository folder not found!"
                    }
                }
            }
        }

        stage('Determine S3 Bucket') {
            steps {
                script {
                    def bucketMap = [
                        'main': 'my-github-backup-bucket-main',
                        'dev': 'my-github-backup-bucket-dev',
                        'test': 'my-github-backup-bucket-test',
                        'prod': 'my-github-backup-bucket-prod'
                    ]
                    
                    if (bucketMap.containsKey(params.BRANCH_NAME)) {
                        env.S3_BUCKET = bucketMap[params.BRANCH_NAME]
                    } else {
                        error("❌ Invalid branch name. No S3 bucket mapped for ${params.BRANCH_NAME}")
                    }
                }
            }
        }

        stage('Sync to S3') {
            steps {
                script {
                    sh """
                    if aws s3 ls "s3://${env.S3_BUCKET}" > /dev/null 2>&1; then
                        aws s3 sync repo s3://${env.S3_BUCKET} --delete
                    else
                        echo "❌ S3 bucket ${env.S3_BUCKET} does not exist!"
                        exit 1
                    fi
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ GitHub repo successfully cloned from branch ${params.BRANCH_NAME} and synced to S3 bucket: ${env.S3_BUCKET}!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs!"
        }
    }
}
