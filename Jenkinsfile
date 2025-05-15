podTemplate(
    label: 'podman-agent',
    containers: [
        containerTemplate(
            name: 'jnlp',
            image: 'mukiwa/jenkins-podman-agent:latest',
            ttyEnabled: true,
            args: '${computer.jnlpmac} ${computer.name}',
            privileged: true
        )
    ]
) {
    node('podman-agent') {
        environment {
            // Define the variables for easy configuration
            DOCKER_REGISTRY = 'docker.io'
            DOCKER_USERNAME = 'mukiwa' // Replace with your Docker Hub username
            IMAGE_NAME = 'simple-website'
            IMAGE_TAG = 'latest'
            FULL_IMAGE_NAME = "${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}" // Ensure this variable is defined
        }

        stage('Clone Repository') {
            checkout([$class: 'GitSCM',
                branches: [[name: '*/main']],
                userRemoteConfigs: [[url: 'https://github.com/Astonie/simple-website.git']]
            ])
        }

        stage('Check Podman Version') {
            sh 'podman --version'
        }

        stage('Build Image') {
            sh 'podman build -t simple-site:latest .'
        }

        stage('Tag Image for Registry') {
            // Tag the image with the correct registry URL
            sh "podman tag simple-site:latest ${FULL_IMAGE_NAME}"
        }

        stage('Login to Docker Registry') {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                // Logging into Docker Hub using the credentials
                sh '''
                    echo "Logging into Docker Hub..."
                    echo $DOCKER_PASS | podman login --username $DOCKER_USER --password-stdin $DOCKER_REGISTRY
                '''
            }
        }

        stage('Push Image to Registry') {
            // Push the image to Docker Hub using the full image name
            sh '''
                echo "Pushing image to Docker Hub..."
                podman push ${FULL_IMAGE_NAME}
            '''
        }

        stage('Deploy to Minikube') {
            // Ensure the kubectl context is set for Minikube (check if kubectl is installed)
            sh '''
                echo "Checking kubectl context..."
                kubectl config get-contexts || exit 1  // List contexts for debugging
                kubectl config use-context minikube || exit 1
                echo "Deploying image to Minikube..."
                kubectl set image deployment/simple-site simple-site=${FULL_IMAGE_NAME} --namespace=default || kubectl apply -f your-kube-deployment.yaml
            '''
        }
    }
}
