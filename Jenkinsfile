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
                envVar(key: 'PATH', value: '/usr/local/bin:$PATH')
            ],
            volumeMounts: [
                // Mount the host's kubeconfig file
                volumeMount(name: 'kubeconfig', mountPath: '/home/jenkins/.kube', readOnly: true)
            ]
        )
    ],
    volumes: [
        // Define the hostPath volume for kubeconfig
        hostPathVolume(hostPath: '/home/jenkins/.kube', mountPath: '/home/jenkins/.kube')
    ]
) {
    node('podman-agent') {
        def DOCKER_REGISTRY = 'docker.io'
        def DOCKER_USERNAME = 'mukiwa'
        def IMAGE_NAME = 'simple-website'
        def K8S_NAMESPACE = 'default'
        def DEPLOYMENT_NAME = 'simple-site'
        def FULL_IMAGE_NAME

        try {
            stage('Clone Repository') {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/Astonie/simple-website.git']]
                ])
                def IMAGE_TAG = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                FULL_IMAGE_NAME = "${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                echo "Full Image Name: ${FULL_IMAGE_NAME}"
            }

            stage('Check Podman Version') {
                def podmanVersion = sh(script: 'podman --version', returnStdout: true).trim()
                echo "Podman Version: ${podmanVersion}"
            }

            stage('Build Image') {
                timeout(time: 10, unit: 'MINUTES') {
                    sh """
                        set -e
                        podman build -t ${FULL_IMAGE_NAME} .
                    """
                }
            }

            stage('Test Image') {
                sh """
                    set -e
                    podman run -d --name test-nginx -p 8080:80 ${FULL_IMAGE_NAME}
                    sleep 5
                    curl http://localhost:8080
                    podman stop test-nginx
                    podman rm test-nginx
                """
            }

            stage('Login to Docker Registry') {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        set -e
                        echo \$DOCKER_PASS | podman login --username \$DOCKER_USER --password-stdin ${DOCKER_REGISTRY}
                    """
                }
            }

            stage('Push Image to Registry') {
                sh """
                    set -e
                    podman push ${FULL_IMAGE_NAME}
                """
            }

            stage('Check Kubectl Config') {
                sh """
                    set -e
                    kubectl config get-contexts
                    kubectl config use-context minikube
                """
            }

            stage('Deploy to Minikube') {
                sh """
                    set -e
                    if ! kubectl get deployment ${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE} > /dev/null 2>&1; then
                        kubectl apply -f your-kube-deployment.yaml
                    else
                        kubectl set image deployment/${DEPLOYMENT_NAME} ${DEPLOYMENT_NAME}=${FULL_IMAGE_NAME} -n ${K8S_NAMESPACE}
                    fi
                    kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE}
                """
            }
        } catch (Exception e) {
            error "Pipeline failed: ${e.message}"
        } finally {
            deleteDir()
        }
    }
}
