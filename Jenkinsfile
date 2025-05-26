pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "veda25/medicure-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Veda-Sr/care.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Test Report') {
            steps {
                sh 'mvn surefire-report:report-only'
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE .
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                docker push $DOCKER_IMAGE
                '''
            }
        }

        stage('Deploy to K8s Test') {
            steps {
                sh 'kubectl apply -f k8s/test-deployment.yaml'
            }
        }

        stage('UI Test with Selenium') {
            steps {
                sh 'python3 selenium_tests.py'
            }
        }

        stage('Deploy to K8s Prod') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh 'kubectl apply -f k8s/prod-deployment.yaml'
            }
        }
    }
}
