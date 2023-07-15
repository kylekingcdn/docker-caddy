pipeline {
    agent any
    environment {
        image_name = 'kylekingcdn/caddy'
        architectures = 'linux/arm64/v8,linux/amd64'
        registry_endpoint = ''
        registry_credentials = 'dockerhub-credentials'
    }
    parameters {
        choice (
            name: 'image_tag',
            description: 'Image tags to build',
            choices: [
                'all',
                'dns',
                'dns-mercure'
            ]
        )
    }
    stages {
        stage('Build dns image') {
            when {
                anyOf {
                    expression { params.image_tag == 'all' }
                    expression { params.image_tag == 'dns' }
                }
            }
            steps {
                script {
                    docker.withRegistry( '', registry_credentials ) {
                        sh label: "Create builder instance", script: """
                            docker buildx create --use
                        """
                        sh label: "Build image", script: """
                            docker buildx build \
                                --file dns.Dockerfile \
                                --tag ${image_name}:dns \
                                --tag ${image_name}:dns-route53 \
                                --tag ${image_name}:dns-digitalocean \
                                --tag ${image_name}:dns-cloudflare \
                                --platform ${architectures} \
                                --force-rm \
                                --no-cache \
                                --pull \
                                --push \
                                .
                        """
                    }
                }
            }
            post {
                cleanup {
                    sh label: "Stop builder", script: """
                        docker buildx stop;
                    """
                    sh label: "Remove builder", script: """
                        docker buildx rm --force;
                    """
                }
            }
        }
        stage('Build dns-mercure image') {
            when {
                anyOf {
                    expression { params.image_tag == 'all' }
                    expression { params.image_tag == 'dns-mercure' }
                }
            }
            steps {
                script {
                    docker.withRegistry( '', registry_credentials ) {
                        sh label: "Create builder instance", script: """
                            docker buildx create --use
                        """
                        sh label: "Build image", script: """
                            docker buildx build \
                                --file dns-mercure.Dockerfile \
                                --tag ${image_name}:dns-mercure \
                                --tag ${image_name}:dns-route53-mercure \
                                --tag ${image_name}:dns-digitalocean-mercure \
                                --tag ${image_name}:dns-cloudflare-mercure \
                                --platform ${architectures} \
                                --force-rm \
                                --no-cache \
                                --pull \
                                --push \
                                .
                        """
                    }
                }
            }
            post {
                cleanup {
                    sh label: "Stop builder", script: """
                        docker buildx stop;
                    """
//                  sh label: "Remove builder", script: """
//                      docker buildx rm --force;
//                  """
                }
            }
        }
    }
    post {
        cleanup {
            sh label: "Run docker buildx prune", script: """
                docker buildx prune --all --force;
            """
            sh label: "Run docker system prune", script: """
                docker system prune --all --force;
            """
            cleanWs()
        }
    }
}
