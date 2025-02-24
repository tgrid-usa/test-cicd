pipeline {
    agent any

    environment {
        REPO_CICD = 'https://github.com/tgrid-usa/test-cicd.git'
        BRANCH = 'fnt-bkt-test-cicd'
        CREDENTIALS_ID = '5bb806d0-f7ec-44bb-bcf9-6194de97138e'
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
    }

    post {
        always {
            cleanWs()
        }
    }
}
