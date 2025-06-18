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
                        // 'sh' được dùng để chạy các lệnh của Linux
                        sh 'echo "--- Checking IP Configuration ---"'
                        sh 'ip addr'
                        
                        sh 'echo "--- Checking listening ports ---"'
                        // Lỗi "command not found" có thể xảy ra nếu net-tools chưa được cài
                        // Nếu lỗi, bạn có thể tạm thời bỏ qua dòng này
                        sh 'netstat -tulpn | grep 9000 || echo "netstat not found, skipping..."'
                        
                        sh 'echo "--- Testing connection to localhost:9000 ---"'
                        // Dùng nc (netcat) để kiểm tra kết nối TCP. 
                        // Lệnh này sẽ báo thành công (succeeded) nếu kết nối được.
                        sh 'nc -z -v localhost 9000'
                        
                        sh 'echo "--- Testing connection to host.docker.internal:9000 ---"'
                        // Thử kết nối bằng curl để xem phản hồi
                        // Cờ --fail sẽ khiến bước này thất bại nếu không kết nối được
                        sh 'curl --fail http://host.docker.internal:9000 || echo "Failed to connect to host.docker.internal"'
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
