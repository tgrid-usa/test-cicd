pipeline {
    agent any

    environment {
        REPO_CICD = 'https://github.com/tgrid-usa/test-cicd.git'
        BRANCH = 'fnt-bkt-test-cicd'
        CREDENTIALS_ID = '5bb806d0-f7ec-44bb-bcf9-6194de97138e'
        GCP_BUCKET = 'gs://testhellotgh'
        GCP_CREDENTIALS_ID = 'gcp-staging'
    }

    stages {
        stage('Get approval') {
            options {
                timeout(time: 59, unit: 'MINUTES')
            }
            steps {
                input "Please Approve To Proceed The Deployment"
            }
        }

        stage('Checkout Latest Code') {
            steps {
                git branch: BRANCH, credentialsId: CREDENTIALS_ID, url: REPO_CICD
                sh 'ls -la'
            }
        }

        stage('Upload to GCP Bucket') {
            steps {
                script {
                    withCredentials([file(credentialsId: GCP_CREDENTIALS_ID, variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh """
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gsutil -m rsync -r . ${GCP_BUCKET}
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
