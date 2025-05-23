FROM jenkins/inbound-agent:latest

USER root

RUN apt-get update && \
    apt-get install -y podman curl ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install kubectl (static version for reliability)
ENV KUBECTL_VERSION=v1.29.0
RUN curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" && \
    chmod +x kubectl && \
    mv kubectl /usr/local/bin/kubectl && \
    kubectl version --client

ENV PATH="/usr/local/bin:$PATH"

USER jenkins
