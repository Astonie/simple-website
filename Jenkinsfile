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

        stage('Build with Podman') {
            sh 'podman build -t simple-site:latest .'
        }

        stage('Run with Podman (Optional)') {
            sh 'podman run -d -p 8080:80 simple-site:latest || true'
        }
    }
// This Jenkinsfile uses a Podman agent to build and run a simple website.
// It includes stages for cloning the repository, checking the Podman version, building the Docker image, and running the container.