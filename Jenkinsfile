pipeline {
    agent any // Chỉ định Jenkins agent sẽ thực thi pipeline

    // Biến môi trường
    environment {
        DOCKER_REGISTRY = 'your-docker-registry' // Ví dụ: '123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app'
        DOCKER_CREDENTIALS = credentials('docker-credentials-id') // ID của credentials trong Jenkins
        KUBE_CONFIG = credentials('kube-config-id') // ID của file kubeconfig trong Jenkins
        SONAR_HOST = 'http://your-sonarqube-server:9000'
        SONAR_TOKEN = credentials('sonarqube-token-id')
        SNYK_TOKEN = credentials('snyk-token-id')
    }

    stages {
        // Giai đoạn 1: Checkout mã nguồn
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/microservice.git'
            }
        }

        // Giai đoạn 2: Build và Test
        stage('Build & Test') {
            steps {
                sh 'mvn clean install' // Build ứng dụng và chạy unit test
            }
        }

        // Giai đoạn 3: Phân tích chất lượng mã nguồn với SonarQube
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

        // Giai đoạn 4 (Tùy chọn): Quét bảo mật với Snyk/Trivy
        stage('Security Scan') {
            steps {
                // Ví dụ với Snyk
                sh "snyk auth ${SNYK_TOKEN}"
                sh "snyk test --all-projects" // Quét các thư viện có lỗ hổng
                sh "snyk container test ${DOCKER_REGISTRY}:${env.BUILD_ID}" // Quét image sau khi build
            }
        }

        // Giai đoạn 5: Build và Push Docker Image
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

        // Giai đoạn 6: Deploy lên Kubernetes
        stage('Deploy to Kubernetes') {
            steps {
                // Sử dụng kubeconfig đã được cấu hình trong Jenkins
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl set image deployment/my-app my-app=${DOCKER_REGISTRY}:${env.BUILD_ID}"
            }
        }
    }

    // Các hành động sau khi pipeline kết thúc
    post {
        always {
            cleanWs() // Dọn dẹp workspace
        }
    }
}
