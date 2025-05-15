podTemplate(
    label: 'podman-agent',
    containers: [
        containerTemplate(
            name: 'jnlp',
            image: 'mukiwa/jenkins-podman-kubectl-agent',
            ttyEnabled: true,
            args: '${computer.jnlpmac} ${computer.name}',
            privileged: true,
            envVars: [
                // Ensure kubectl is in the path
                [key: 'PATH', value: '/usr/local/bin:$PATH']
            ]
        )
    ]
) {
    node('podman-agent') {
        // ✅ Use regular Groovy variable declarations here
        def DOCKER_REGISTRY = 'docker.io'
        def DOCKER_USERNAME = 'mukiwa'
        def IMAGE_NAME = 'simple-website'
        def IMAGE_TAG = 'latest'
        def FULL_IMAGE_NAME = "${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"

        def K8S_NAMESPACE = 'default'
        def DEPLOYMENT_NAME = 'simple-site'

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
            sh """
                echo "Building container image..."
                podman build -t ${FULL_IMAGE_NAME} .
            """
        }

        stage('Login to Docker Registry') {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                sh """
                    echo "Logging into Docker Hub..."
                    echo \$DOCKER_PASS | podman login --username \$DOCKER_USER --password-stdin ${DOCKER_REGISTRY}
                """
            }
        }

        stage('Push Image to Registry') {
            sh """
                echo "Pushing image to Docker Hub..."
                podman push ${FULL_IMAGE_NAME}
            """
        }

        stage('Deploy to Minikube') {
            sh """
                echo "Deploying image to Minikube..."
                kubectl config use-context minikube || exit 1
                kubectl set image deployment/${DEPLOYMENT_NAME} ${DEPLOYMENT_NAME}=${FULL_IMAGE_NAME} --namespace=${K8S_NAMESPACE} || kubectl apply -f your-kube-deployment.yaml
            """
        }
    }
}
