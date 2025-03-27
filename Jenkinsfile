pipeline {
    agent any

    parameters {
        choice(name: 'ENV', choices: ['dev', 'test', 'prod'], description: 'Select the target environment')
    }

    environment {
        S3_BUCKET = "my-github-backup-bucket-${params.ENV}"
        REPO_URL = "https://github.com/ishanpathak98/S3-Test.git"
        BRANCH = "main"
        LOCAL_REPO = "repo"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh """
                        rm -rf ${LOCAL_REPO}
                        git clone -b ${BRANCH} ${REPO_URL} ${LOCAL_REPO}
                    """
                }
            }
        }

        stage('Verify Repository') {
            steps {
                script {
                    if (!fileExists("${LOCAL_REPO}")) {
                        error "❌ Repository clone failed!"
                    }
                }
            }
        }

        stage('Check S3 Bucket') {
            steps {
                script {
                    def bucketExists = sh(script: "aws s3api head-bucket --bucket ${S3_BUCKET} 2>&1 || true", returnStdout: true).trim()
                    if (bucketExists.contains("Not Found")) {
                        error "❌ S3 bucket ${S3_BUCKET} does not exist!"
                    } else if (bucketExists.contains("Forbidden")) {
                        error "❌ Access denied to S3 bucket ${S3_BUCKET}! Check IAM permissions."
                    }
                }
            }
        }

        stage('Sync to S3') {
            steps {
                script {
                    sh """
                        aws s3 sync ${LOCAL_REPO} s3://${S3_BUCKET}/ --delete
                        echo "✅ Sync to S3 (${params.ENV}) completed successfully!"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully for ${params.ENV}!"
        }
        failure {
            echo "❌ Pipeline failed for ${params.ENV}. Check logs!"
        }
    }
}
