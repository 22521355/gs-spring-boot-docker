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

                stage('SonarQube Analysis') {
                    steps {
                        // Tên 'MySonarQubeServer' vẫn giữ nguyên vì nó tham chiếu đến cấu hình trong Jenkins
                        withSonarQubeEnv('MySonarQubeServer') { 
                            dir('complete') {
                                // Sử dụng 3 dấu ngoặc kép """ để viết lệnh trên nhiều dòng cho dễ đọc
                                sh """
                                    mvn sonar:sonar \
                                      -Dsonar.projectKey=22521355-1 \
                                      -Dsonar.organization=22521355 \
                                      -Dsonar.host.url=https://sonarcloud.io
                                """
                                // Lưu ý: Không cần -Dsonar.login=... nữa vì withSonarQubeEnv đã lo việc đó
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
