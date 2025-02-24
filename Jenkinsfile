pipeline {
    agent any

    environment {
        REPO_CICD = 'https://github.com/tgrid-usa/test-cicd.git'
        BRANCH = 'fnt-bkt-test-cicd'
        CREDENTIALS_ID = '5bb806d0-f7ec-44bb-bcf9-6194de97138e'
        GCP_BUCKET = 'gs://testhellotgh'
        GCP_CREDENTIALS_ID = 'gcp-staging'
        CLONE_DIR = 'tg-fnt-bkt-test-cicd'
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
                sh "ls -la"
            }
        }

        stage('Install Dependencies and Build') {
            steps {
                dir(CLONE_DIR) {
                    sh "yarn"
                    sh "yarn build"
                }
            }
        }

        stage('Check GCP Bucket Existence') {
            steps {
                script {
                    def bucketExists = sh(script: "gsutil ls ${GCP_BUCKET}", returnStatus: true)
                    if (bucketExists != 0) {
                        error "Bucket '${GCP_BUCKET}' does not exist. Please create the bucket."
                    }
                }
            }
        }

        stage('Upload to GCP Bucket') {
            steps {
                withCredentials([file(credentialsId: GCP_CREDENTIALS_ID, variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                    gsutil -m rsync -r -x "(\\.git|README.md|Jenkinsfile)" . ${GCP_BUCKET}
                    '''
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
