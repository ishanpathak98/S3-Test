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
                    def repoExists = sh(script: "[ -d repo ] && echo 'exists' || echo 'not found'", returnStdout: true).trim()
                    if (repoExists != 'exists') {
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
                        withEnv(["S3_BUCKET=${bucketMap[params.BRANCH_NAME]}"]) {
                            echo "✅ S3 Bucket set to ${env.S3_BUCKET}"
                        }
                    } else {
                        error("❌ Invalid branch name. No S3 bucket mapped for ${params.BRANCH_NAME}")
                    }
                }
            }
        }

        stage('Sync to S3') {
            steps {
                script {
                    withEnv(["S3_BUCKET=${env.S3_BUCKET}"]) {
                        def s3Exists = sh(script: "aws s3 ls s3://${env.S3_BUCKET} > /dev/null 2>&1 && echo 'exists' || echo 'not found'", returnStdout: true).trim()
                        if (s3Exists != 'exists') {
                            error "❌ S3 bucket ${env.S3_BUCKET} does not exist!"
                        }

                        sh "aws s3 sync repo s3://${env.S3_BUCKET} --delete"
                    }
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
