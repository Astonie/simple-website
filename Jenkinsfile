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
            // Docker registry details
            DOCKER_REGISTRY = 'docker.io'
            DOCKER_USERNAME = 'mukiwa' // Replace with your Docker Hub username
            IMAGE_NAME = 'simple-website'
            IMAGE_TAG = 'latest'
            FULL_IMAGE_NAME = "${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"

            // Minikube/Kubernetes details
            K8S_NAMESPACE = 'default'
            DEPLOYMENT_NAME = 'simple-site'
        }

        // Stage to clone the repository
        stage('Clone Repository') {
            checkout([$class: 'GitSCM',
                branches: [[name: '*/main']],
                userRemoteConfigs: [[url: 'https://github.com/Astonie/simple-website.git']]
            ])
        }

        // Stage to check Podman version
        stage('Check Podman Version') {
            sh 'podman --version'
        }

        // Stage to build the container image
        stage('Build Image') {
            sh '''
                echo "Building container image..."
                podman build -t ${IMAGE_NAME}:${IMAGE_TAG} .
            '''
        }

        // Stage to tag the image with full registry name
        stage('Tag Image for Registry') {
            sh "podman tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}"
        }

        // Stage to log into Docker registry
        stage('Login to Docker Registry') {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                sh '''
                    echo "Logging into Docker Hub..."
                    echo $DOCKER_PASS | podman login --username $DOCKER_USER --password-stdin $DOCKER_REGISTRY
                '''
            }
        }

        // Stage to push the image to Docker registry
        stage('Push Image to Registry') {
            sh '''
                echo "Pushing image to Docker Hub..."
                podman push ${FULL_IMAGE_NAME}
            '''
        }

        // Stage to deploy to Minikube
        stage('Deploy to Minikube') {
            // Ensure the kubectl context is set for Minikube
            sh '''
                echo "Checking kubectl context..."
                kubectl config get-contexts || exit 1  // List contexts for debugging
                kubectl config use-context minikube || exit 1
                echo "Deploying image to Minikube..."
                kubectl set image deployment/${DEPLOYMENT_NAME} ${DEPLOYMENT_NAME}=${FULL_IMAGE_NAME} --namespace=${K8S_NAMESPACE} || kubectl apply -f your-kube-deployment.yaml
            '''
        }
    }
}
