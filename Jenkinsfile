pipeline {
    agent none

    stages {
        stage('Pipeline Execution') {
            agent any

            environment {
                DOCKER_REGISTRY = 'your-docker-registry22521355/gs-spring-boot-docker'
                DOCKER_CREDENTIALS = credentials('docker-credentials-id')
                KUBE_CONFIG = credentials('kube-config-id') //
                SONAR_TOKEN = credentials('sonarqube-api-token')
                SNYK_TOKEN = credentials('snyk-token-id') //
            }

            tools {
                maven 'Maven-3.9.6'
            }
            
            stages {
                stage('Checkout') {
                    steps {
                        git url: 'https://github.com/22521355/gs-spring-boot-docker.git', branch: 'main'}
                }

                stage('Build & Test') {
                    steps {
                        dir('complete') {
                            sh 'mvn clean install'
                        }
                    }
                }

                stage('Debug Network') {
                    steps {
                        // bat' được dùng để chạy các lệnh của Command Prompt trên Windows
                        bat 'echo "--- Checking IP Configuration ---"'
                        bat 'ipconfig'
                        
                        bat 'echo "--- Checking listening ports ---"'
                        bat 'netstat -an | find "9000"'
                        
                        // Sử dụng PowerShell để kiểm tra kết nối mạng
                        bat 'echo "--- Testing connection to localhost:9000 ---"'
                        powershell 'Test-NetConnection -ComputerName localhost -Port 9000'
                        
                        bat 'echo "--- Testing connection to host.docker.internal:9000 ---"'
                        powershell 'Test-NetConnection -ComputerName host.docker.internal -Port 9000'
                    }
                }
                
                stage('SonarQube Analysis') {
                    steps {
                        dir('complete') {
                        withSonarQubeEnv('MySonarQubeServer') {
                            sh "mvn sonar:sonar \
                                -Dsonar.projectKey=my-microservice \
                                -Dsonar.host.url=http://host.docker.internal:9000 \
                                -Dsonar.login=${SONAR_TOKEN}"
                        }
                        }
                    }
                }

                stage('Security Scan') {
                    steps {
                        // Ví dụ với Snyk
                        sh "snyk auth ${SNYK_TOKEN}"
                        sh "snyk test --all-projects" // Quét các thư viện có lỗ hổng
                        sh "snyk container test ${DOCKER_REGISTRY}:${env.BUILD_ID}" // Quét image sau khi build
                    }
                }

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
    }
}
