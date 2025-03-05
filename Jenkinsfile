pipeline {
    agent any

    environment {
        REPO_CICD = 'https://github.com/tgrid-usa/test-cicd.git'
        BRANCH = 'fnt-bkt-test-cicd'
        CREDENTIALS_ID = '5bb806d0-f7ec-44bb-bcf9-6194de97138e'
        GCP_BUCKET = 'gs://testhellotgh'
        GCP_CREDENTIALS_ID = 'gcp-staging'
        GCP_REGION = 'asia-south1'
        GCP_PROJECT = 'tg-uat-446010'
        CLONE_DIR = 'tg-fnt-bkt-test-cicd'
        SONAR_PROJECT_KEY = 'TG-Test-CICD'
        SONAR_PROJECT_NAME = 'Issuerss-TG-Test-CICD'
        SONAR_ORG = 'trustgrid-staging'
        SONAR_URL = 'https://sonarcloud.io'
        SONAR_LANGUAGE = 'js'
        GCP_SECRET_NAME = 'Test-CICD-Secret'
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
                script {
                    if (!fileExists(CLONE_DIR)) {
                        sh "mkdir -p ${CLONE_DIR}"
                    }
                }
                dir(CLONE_DIR) {
                    git branch: BRANCH, credentialsId: CREDENTIALS_ID, url: REPO_CICD
                    sh "ls -la"
                }
            }
        }

        stage('Create .env from GCP Secret Manager') {
            steps {
                withCredentials([file(credentialsId: GCP_CREDENTIALS_ID, variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    dir(CLONE_DIR) {
                        sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        SECRET_VALUE=$(gcloud secrets versions access latest --secret="$GCP_SECRET_NAME" --project="$GCP_PROJECT")
                        echo "$SECRET_VALUE" > .env
                        cat .env
                        '''
                    }
                }
            }
        }

        stage('Install Dependencies and Build') {
            steps {
                dir(CLONE_DIR) {
                    sh "yarn install --frozen-lockfile"
                    sh "yarn build"
                    sh "ls -la"
                }
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarcloud', variable: 'SONAR_TOKEN')]) {
                    dir(CLONE_DIR) {
                        sh """
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.organization=${SONAR_ORG} \
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_URL} \
                        -Dsonar.token=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Check GCP Bucket Existence') {
            steps {
                script {
                    def bucketExists = sh(script: "gsutil ls ${GCP_BUCKET} >/dev/null 2>&1", returnStatus: true)
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
                    gsutil -m rsync -r -x "(\\.git|README.md|Jenkinsfile)" ${CLONE_DIR}/build/ ${GCP_BUCKET}/
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
