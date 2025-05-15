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
            sh 'podman tag simple-site:latest docker.io/your-docker-username/simple-site:latest'
        }

        stage('Login to Docker Registry') {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                sh 'podman login -u $DOCKER_USER -p $DOCKER_PASS docker.io'
            }
        }

        stage('Push Image to Registry') {
            sh 'podman push docker.io/mukiwa/simple-site:latest'
        }

        stage('Deploy to Minikube') {
            // Make sure your kubeconfig is available and kubectl is installed in the agent
            sh 'kubectl set image deployment/simple-site simple-site=docker.io/mukiwa/simple-site:latest --namespace=default || kubectl apply -f your-kube-deployment.yaml'
        }
    }
}

// This Jenkinsfile builds, pushes, and deploys a container image using Podman and Minikube.