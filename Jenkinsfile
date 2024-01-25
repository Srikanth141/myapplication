pipeline {
    agent any
    tools{
        jdk  'JDK'
        maven 'Maven'
    }
    environment{
        SCANNER_HOME=tool 'SonarQube Scanner'
        APP_NAME = "application"
        RELEASE = "1.0.0"
        DOCKER_USER = "srikanthmadhavarapu"
        DOCKER_PASS = "Docker-hub"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        APP_NAME = "application"
        JENKINS_API_TOKEN = credentials{'JENKINS_API_TOKEN'}
    }

    stages {
        stage('cleanws') {
            steps {
                cleanWs()
            }
        }
        stage('git clone') {
            steps {
                git branch: 'main', credentialsId: 'Git-hub', url: 'https://github.com/Srikanth141/complete-prodcution-e2e-pipeline.git'
            }
        }
        stage('Maven Build') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('Maven Test') {
            steps {
                sh "mvn test"
            }
        }
        stage('Sonarqube analysis') {
            steps {
                script{
                    withSonarQubeEnv('SonarQube') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
        stage('Sonarquality gates') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        stage('Trivy filescan') {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage('Docker build and push') {
            steps {
                script{
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage('Trivy image-scan') {
            steps {
                script{
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image srikanthmadhavarapu/application:1.0.0-5 --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }
        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-127-212-16.ap-south-1.compute.amazonaws.com:8080/job/Srikanth-CD/buildWithParameters?token=gitops-token'"
                }
            }
        }
    }
}
