pipeline {
    agent none

    environment {
        DOCKER_REGISTRY = 'your-docker-registry22521355/gs-spring-boot-docker'
        DOCKER_CREDENTIALS = credentials('docker-credentials-id')
        KUBE_CONFIG = credentials('kube-config-id') //
        SONAR_TOKEN = credentials('squ_61d7f0aafd439902e9305d2a18f2e8e808c53fdc')
        SNYK_TOKEN = credentials('snyk-token-id') //
    }

    stages {
        stage('Pipeline Execution') {
            agent any

            stages {
                stage('Checkout') {
                    steps {
                        git 'https://github.com/22521355/gs-spring-boot-docker.git'
                    }
                }

                stage('Build & Test') {
                    steps {
                        sh 'mvn clean install'
                    }
                }

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
