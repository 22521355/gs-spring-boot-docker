pipeline {
    agent none

    stages {
        stage('Pipeline Execution') {
            agent any

            environment {
                DOCKER_REGISTRY = '22521355/gs-spring-boot-docker'
                //DOCKER_CREDENTIALS = credentials('docker-credentials-id')
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
                            //tạo ra các thư mục BOOT-INF, META-INF mà Dockerfile cần
                            sh 'mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)'
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
                                      -Dsonar.organization=22521355-1 \
                                      -Dsonar.host.url=https://sonarcloud.io
                                """
                            }
                        }
                    }
                }
                
                stage('Security Scan') {
                    steps {
                        dir('complete') {
                            withCredentials([string(credentialsId: 'snyk-token-id', variable: 'SNYK_SECRET_TOKEN')]) {                     
                                sh 'echo "Installing Snyk CLI..."'
                                sh 'curl -Lo ./snyk https://static.snyk.io/cli/latest/snyk-linux'
                                sh 'chmod +x ./snyk'
                                sh './snyk auth $SNYK_SECRET_TOKEN'
                                //bỏ qua lỗi
                                sh './snyk test --all-projects || true'
                            }
                        }
                    }
                }

                stage('Build & Push Docker Image') {
                    steps {
                        script {
                            dir('complete') { 
                                def dockerImage = docker.build("${DOCKER_REGISTRY}:${env.BUILD_ID}", ".")
                                docker.withRegistry("https://hub.docker.com", 'docker-credentials-id') {
                                    dockerImage.push()
                                }
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
