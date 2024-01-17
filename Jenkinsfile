pipeline {
    agent any
    environment {
        image_name = 'kylekingcdn/caddy'
        architectures = 'linux/arm64,linux/amd64'
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
                        builder_name = env.architectures.replaceAll(',', '-').replaceAll('/', '_')
                        sh label: "Create a new builder if one does not already exist", script: """
                            docker buildx inspect ${builder_name} || docker buildx create --name ${builder_name} --platform ${architectures}
                        """
                        sh label: "Build image", script: """
                            docker buildx build \
                                --file dns.Dockerfile \
                                --tag ${image_name}:dns \
                                --tag ${image_name}:dns-route53 \
                                --tag ${image_name}:dns-digitalocean \
                                --tag ${image_name}:dns-cloudflare \
                                --platform ${architectures} \
                                --builder ${builder_name} \
                                --force-rm \
                                --no-cache \
                                --pull \
                                --push \
                                .
                        """
                    }
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
                        builder_name = env.architectures.replaceAll(',', '-').replaceAll('/', '_')
                        sh label: "Create a new builder if one does not already exist", script: """
                            docker buildx inspect ${builder_name} || docker buildx create --name ${builder_name} --platform ${architectures}
                        """
                        sh label: "Build image", script: """
                            docker buildx build \
                                --file dns-mercure.Dockerfile \
                                --tag ${image_name}:dns-mercure \
                                --tag ${image_name}:dns-route53-mercure \
                                --tag ${image_name}:dns-digitalocean-mercure \
                                --tag ${image_name}:dns-cloudflare-mercure \
                                --platform ${architectures} \
                                --builder ${builder_name} \
                                --force-rm \
                                --no-cache \
                                --pull \
                                --push \
                                .
                        """
                    }
                }
            }
        }
    }
    post {
        cleanup {
            cleanWs()
        }
    }
}
