pipeline {
    agent any

    tools {
        jdk 'JDK11'          // Jenkins-managed JDK
        maven 'Maven3'       // Jenkins-managed Maven
        sonar 'SonarQube Scanner'  // Jenkins-managed SonarQube Scanner
    }

    stages {
        stage('Checkout') {
            steps {
                sh 'echo passed'
            }
        }

        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                // build the project and create a JAR file
                sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn clean package'
            }
        }

        stage('Static Code Analysis') {
            steps {
                // Ensure SonarQube authentication token is available
                withCredentials([string(credentialsId: 'SonarQube', variable: 'SONAR_AUTH_TOKEN')]) {
                    withSonarQubeEnv('SonarQube Scanner') {  // Use the SonarQube server defined in Jenkins
                        sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN'
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "romilsingal2024/ultimate-cicd:${BUILD_NUMBER}"
                REGISTRY_CREDENTIALS = credentials('docker-cred')
            }
            steps {
                script {
                    sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "Jenkins-Zero-To-Hero"
                GIT_USER_NAME = "romilsingal23"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "romilsingal2024@gmail.com"
                        git config user.name "Romil Singal"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        sed -i "s/ultimate-cicd:.*/ultimate-cicd:${BUILD_NUMBER}/g" java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                        git add java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
