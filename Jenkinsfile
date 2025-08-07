pipeline {
    agent any
    
    environment {
        S3_BUCKET = 'mersch0823'
        SOURCE_FOLDER = 'louisejean'
        AWS_DEFAULT_REGION = 'us-east-1'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
                sh 'pwd'
                sh 'ls -la'
            }
        }
        
        stage('Verify Source Folder') {
            steps {
                echo 'Verifying source folder exists...'
                script {
                    if (fileExists("${SOURCE_FOLDER}")) {
                        echo "✅ Source folder '${SOURCE_FOLDER}' found"
                        sh "ls -la ${SOURCE_FOLDER}/"
                    } else {
                        error "❌ Source folder '${SOURCE_FOLDER}' not found in repository"
                    }
                }
            }
        }
        
        stage('Test AWS Access') {
            steps {
                echo 'Testing AWS S3 access...'
                sh '''
                    aws sts get-caller-identity
                    aws s3 ls s3://${S3_BUCKET}/ || echo "Bucket might be empty or access issues"
                '''
            }
        }
        
        stage('Upload to S3') {
            steps {
                echo 'Uploading files to S3...'
                sh '''
                    cd ${SOURCE_FOLDER}
                    echo "Files to upload:"
                    find . -type f
                    aws s3 cp . s3://${S3_BUCKET}/ --recursive --exclude "*.git*"
                    echo "Verifying upload..."
                    aws s3 ls s3://${S3_BUCKET}/ --recursive
                '''
            }
        }
    }
    
    post {
        success {
            echo '🎉 Pipeline completed successfully!'
            echo "Files from '${SOURCE_FOLDER}' have been uploaded to S3 bucket '${S3_BUCKET}'"
        }
        failure {
            echo '❌ Pipeline failed!'
            echo 'Check the logs above for error details'
        }
        always {
            echo 'Pipeline execution completed.'
            cleanWs()
        }
    }
}
