pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Enter the branch name (e.g., feature/master)')
        choice(name: 'ENV', choices: ['prod', 'stage', 'dev', 'test'], description: 'Select Environment')
    }

    environment {
        BUCKET_NAME = "my-github-backup-bucket-test"  // FIXED: Single bucket for all environments
        AWS_REGION = "us-east-1"
        S3_PATH = "${params.ENV}/files-folder/"  // Store files under the selected environment
        REPO_URL = "https://github.com/ishanpathak98/S3-Test.git"
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    sh "rm -rf repo"
                    sh "git clone -b ${params.BRANCH} ${REPO_URL} repo"
                }
            }
        }

        stage('Verify Repository') {
            steps {
                script {
                    if (!fileExists('repo')) {
                        error "❌ Cloning failed. Repository not found!"
                    }
                }
            }
        }

        stage('Check S3 Bucket') {
            steps {
                script {
                    def bucketExists = sh(script: "aws s3api head-bucket --bucket ${BUCKET_NAME}", returnStatus: true)
                    if (bucketExists != 0) {
                        error "❌ S3 bucket ${BUCKET_NAME} does not exist!"
                    }
                }
            }
        }

        stage('Create Folder in S3') {
            steps {
                script {
                    echo "Creating folder structure in S3: s3://${BUCKET_NAME}/${S3_PATH}"
                    sh "aws s3api put-object --bucket ${BUCKET_NAME} --key ${S3_PATH}"
                }
            }
        }

        stage('Sync Files to S3') {
            steps {
                script {
                    echo "Uploading files to S3 bucket: s3://${BUCKET_NAME}/${S3_PATH}"
                    sh "aws s3 sync repo/ s3://${BUCKET_NAME}/${S3_PATH} --region ${AWS_REGION}"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Successfully synced ${params.BRANCH} branch to S3: s3://${BUCKET_NAME}/${S3_PATH}"
        }
        failure {
            echo "❌ Pipeline failed for ${params.ENV}. Check logs!"
        }
    }
}
