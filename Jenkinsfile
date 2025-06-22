pipeline {
    agent any // Run on any available Jenkins agent

    environment {
        // Jenkins Credential ID for Docker Hub
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-credentials' // MUST match the ID you set in Jenkins

        // Your Docker Hub username
        DOCKER_HUB_USERNAME = 'sampathreddy926' // REPLACE THIS! e.g., 'myusername'

        // Name for your Docker image
        DOCKER_IMAGE_NAME = 'wedding-website' // You can change this name

        // Tag for the image (uses Jenkins build number for uniqueness)
        DOCKER_TAG = "${env.BUILD_NUMBER}"

        // Full Docker Hub repository path
        DOCKER_HUB_REPO = "${DOCKER_HUB_USERNAME}/${DOCKER_IMAGE_NAME}"
    }

    stages {
        stage('1. Checkout Code') {
            steps {
                echo "--- Stage: Checking out code from GitHub ---"
                // Jenkins automatically checks out the code when using "Pipeline script from SCM"
            }
        }

        stage('2. Build Docker Image') {
            steps {
                script {
                    echo "--- Stage: Building Docker Image ---"
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_HUB_CREDENTIALS_ID}") {
                        def customImage = docker.build "${DOCKER_HUB_REPO}:${DOCKER_TAG}", '.'
                        echo "Docker Image built: ${customImage.id}"
                    }
                }
            }
        }

        stage('3. Run and Test Container (Optional)') {
            steps {
                script {
                    echo "--- Stage: Running temporary container for testing ---"
                    // Run container, map port 8080 on host to 80 in container
                    sh "docker run -d -p 8096:80 --name temp-website-container-${env.BUILD_NUMBER} ${DOCKER_HUB_REPO}:${DOCKER_TAG}"
                    sleep 10 // Give time for Nginx to start
                    sh "curl -f http://localhost:8096 || { echo 'ERROR: Website not accessible!'; exit 1; }"
                    sh "docker stop temp-website-container-${env.BUILD_NUMBER} || true"
                    sh "docker rm temp-website-container-${env.BUILD_NUMBER} || true"
                    echo "Temporary container tested and cleaned up."
                }
            }
        }

        stage('4. Push Image to Docker Hub') {
            steps {
                script {
                    echo "--- Stage: Pushing Image to Docker Hub ---"
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_HUB_CREDENTIALS_ID}") {
                        def customImage = docker.image("${DOCKER_HUB_REPO}:${DOCKER_TAG}")
                        customImage.push() // Push with build number tag
                        customImage.push("latest") // Also push with 'latest' tag
                        echo "Image pushed to Docker Hub: ${DOCKER_HUB_REPO}:${DOCKER_TAG} and :latest"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        success {
            echo 'SUCCESS: Pipeline completed!'
        }
        failure {
            echo 'FAILURE: Pipeline failed! Check logs.'
        }
    }
}