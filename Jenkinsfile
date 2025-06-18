pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'your-docker-registry22521355/gs-spring-boot-docker'
        DOCKER_CREDENTIALS = credentials('docker-credentials-id')
        KUBE_CONFIG = credentials('kube-config-id') //
        SONAR_TOKEN = credentials('squ_61d7f0aafd439902e9305d2a18f2e8e808c53fdc')
        SNYK_TOKEN = credentials('snyk-token-id') //
    }

    stages {
        //Checkout mã nguồn
        stage('Checkout') {
            steps {
                git 'https://github.com/22521355/gs-spring-boot-docker.git'
            }
        }

        //Build và Test
        stage('Build & Test') {
            steps {
                sh 'mvn clean install' 
            }
        }

        //Phân tích chất lượng mã nguồn với SonarQube
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh "mvn sonar:sonar \
                        -Dsonar.projectKey=my-microservice \
                        -Dsonar.host.url=${SONAR_HOST} \
                        -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        //Quét bảo mật với Snyk
        stage('Security Scan') {
            steps {
                sh "snyk auth ${SNYK_TOKEN}"
                sh "snyk test --all-projects" 
                sh "snyk container test ${DOCKER_REGISTRY}:${env.BUILD_ID}" 
            }
        }

        //Build và Push Docker Image
        stage('Build & Push Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build("${DOCKER_REGISTRY}:${env.BUILD_ID}", ".")
                    docker.withRegistry("https://${DOCKER_REGISTRY}", DOCKER_CREDENTIALS) {
                        dockerImage.push()
                    }
                }
            }
        }

        //Deploy lên Kubernetes
        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl set image deployment/my-app my-app=${DOCKER_REGISTRY}:${env.BUILD_ID}"
            }
        }
    }

    post {
        cleanup {
            cleanWs()
        }
    }
}
