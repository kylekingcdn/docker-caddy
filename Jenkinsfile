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
                'dns-route53',
                'dns-route53-mercure',
                'dns-digitalocean',
                'dns-digitalocean-mercure'
            ]
        )
    }
    stages {
        stage('Build') {
            steps {
                script {
                    docker.withRegistry( '', registry_credentials ) {
                        def tags = []
                        if (params.image_tag == 'all') {
                            tags = [ 'dns-route53', 'dns-route53-mercure', 'dns-digitalocean', 'dns-digitalocean-mercure' ]
                        }
                        else {
                            tags = [ params.image_tag ]
                        }

                        for (int i = 0; i < tags.size(); ++i) {
                            def tag = tags[i]
                            sh label: "Build ${tag}", script: """
                                docker buildx create \
                                    --use && \
                                docker buildx build \
                                    --file ${tag}.Dockerfile \
                                    --tag ${image_name}:${tag} \
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
            }
        }
    }
    post {
        cleanup {
            sh label: "Prune images ", script: """
                docker image prune --force;
            """
            cleanWs()
        }
    }
}
