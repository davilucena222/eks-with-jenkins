pipeline {
    agent any

    environment {
        IMAGE_NAME = 'davilucena/jenkins-flask-app'
        IMAGE_TAG = "${IMAGE_NAME}:${env.GIT_COMMIT}"
        KUBECONFIG = credentials('kubeconfig-credentials-id')
        // AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        // AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')  
    }

    
    stages {
        stage('Setup') {
            steps {
                sh 'whoami'
                sh 'ls -la $KUBECONFIG'
                sh 'chmod 644 $KUBECONFIG'
                sh 'ls -la $KUBECONFIG'
                sh "pip install --no-cache-dir --break-system-packages -r requirements.txt"
            }
        }
        stage('Test') {
            steps {
                sh "pytest"
            }
        }

        stage('Login to docker hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                sh 'echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin'}
                echo 'Login successfully'
            }
        }

        stage('Build Docker Image')
        {
            steps
            {
                sh 'docker build -t ${IMAGE_TAG} .'
                echo "Docker image build successfully"
                sh 'docker image ls'
                
            }
        }

        stage('Push Docker Image')
        {
            steps
            {
                sh 'docker push ${IMAGE_TAG}'
                echo "Docker image push successfully"
            }
        }

        stage('Deploy to Staging')
        {
            steps {
                sh 'sudo kubectl config use-context staging'
                sh 'sudo kubectl config current-context'
                sh "sudo kubectl set image deployment/flask-app flask-app=${IMAGE_TAG}"
            }
        }

        stage('Acceptance Test')
        {
            steps {

                script {

                    def service = sh(script: "kubectl get svc flask-app-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}:{.spec.ports[0].port}'", returnStdout: true).trim()
                    echo "${service}"

                    sh "k6 run -e SERVICE=localhost${service} acceptance-test.js"
                }
            }
        }
        stage('Deploy to Prod')
        {
            steps {
                sh 'sudo kubectl config use-context prod'
                sh 'sudo kubectl config current-context'
                sh "sudo kubectl set image deployment/flask-app flask-app=${IMAGE_TAG}"
            }
        }       

        
    }
}