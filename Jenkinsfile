pipeline {
    agent any

    parameters {
        choice(name: 'ENV', choices: ['prod', 'dev', 'test'], description: 'Select Environment')
    }

    environment {
        BUCKET_NAME = "my-github-backup-bucket-${params.ENV}"
        AWS_REGION = "us-east-1"
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    // Clone the GitHub repo based on the selected branch
                    sh "rm -rf repo"
                    sh "git clone -b ${params.ENV} https://github.com/ishanpathak98/S3-Test.git repo"
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

        stage('Sync to S3') {
            steps {
                script {
                    sh "aws s3 sync repo/ s3://${BUCKET_NAME}/ --region ${AWS_REGION}"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Successfully synced to S3: ${BUCKET_NAME}"
        }
        failure {
            echo "❌ Pipeline failed for ${params.ENV}. Check logs!"
        }
    }
}
