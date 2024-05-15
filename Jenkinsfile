pipeline {
    agent any
    
    tools{
        nodejs 'node'
    }

    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('gitcheckout') {
            steps {
                git changelog: false, poll: false, url: 'https://github.com/singhrajrohit/Doctor-Appointment-react-js.git'
            }
        }
        stage('install dependecies') {
            steps {
                sh 'npm install'
            }
        }
        
    /*stage('Build') {
            steps {
                sh 'npm run build'
            }
        }*/
stage('Sonar Code Analysis') {
            environment {
                SONAR_URL = "http://57.180.65.82:9000"
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    script {
                        def scannerHome = tool 'SonarQubeScanner'  // Ensure SonarQube Scanner tool is configured in Jenkins
                        sh "${scannerHome}/bin/sonar-scanner " +
                           "-Dsonar.projectKey=Doctor-Appointment-react-js " +
                           "-Dsonar.sources=. " +
                           "-Dsonar.host.url=${SONAR_URL} " +
                           "-Dsonar.login=${SONAR_AUTH_TOKEN}"
                    }
                }
            }
        }
    
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "singhrajrohit/doctor-appointment:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
    }
      steps {
        script {
            sh ' docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
    }
 }
    stage('Pull and Deploy'){
        environment {
        DOCKER_IMAGE = "singhrajrohit/doctor-appointment:${BUILD_NUMBER}"
        CONTAINER_NAME = "doctor-appointment-app"
        PORT_MAPPING = "2024:3000" // Adjust port mapping as needed
    }
            steps {
                script {
                    // Pull the Docker image
                    sh "docker pull ${DOCKER_IMAGE}"
                    
                    // Stop and remove any existing container with the same name
                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm ${CONTAINER_NAME} || true"
                    
                    // Run the container
                    sh "docker run -d --name ${CONTAINER_NAME} -p ${PORT_MAPPING} ${DOCKER_IMAGE}"
                    
                    // Optionally, you may want to add additional steps here, such as health checks or testing
                }
            }
        }
    }
}
