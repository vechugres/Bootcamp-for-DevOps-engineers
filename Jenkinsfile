pipeline {
    agent any

    stages {
        stage('Validate Dockerfile') {
            steps {
                sh 'docker run --rm -i hadolint/hadolint < Dockerfile'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build('word-cloud-generator')
                }
            }
        }

        stage('Test Image') {
            steps {
                script {
                    docker.withRegistry('https://hub.docker.com/', 'credentials-id') {
                        docker.image('word-cloud-generator').push()
                    }
                }
            }
        }

        stage('Deploy to Pre-Production') {
            steps {
                script {
                    kubernetesDeploy(
                        kubeconfigId: 'my-kubeconfig',
                        configs: 'k8s/pre-production/deployment.yaml',
                        enableConfigSubstitution: true,
                        namespace: 'pre-production'
                    )
                }
            }
        }

        stage('Test Pre-Production Deployment') {
            steps {
                script {
                    def deploymentStatus = kubernetesStatus(
                        kubeconfigId: 'my-kubeconfig',
                        configs: 'k8s/pre-production/deployment.yaml',
                        enableConfigSubstitution: true,
                        namespace: 'pre-production'
                    )

                    if (deploymentStatus == 'Successful') {
                        input message: 'Pre-Production deployment was successful. Proceed with Production deployment?', ok: 'Deploy to Production'
                    } else {
                        error "Pre-Production deployment failed with status: ${deploymentStatus}"
                    }
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    kubernetesDeploy(
                        kubeconfigId: 'my-kubeconfig',
                        configs: 'k8s/production/deployment.yaml.yaml',
                        enableConfigSubstitution: true,
                        namespace: 'production'
                    )
                }
            }
        }

        stage('Clean Pre-Production Deployment') {
            steps {
                script {
                    kubernetesDelete(
                        kubeconfigId: 'my-kubeconfig',
                        configs: 'k8s/pre-production/deployment.yaml',
                        enableConfigSubstitution: true,
                        namespace: 'pre-production'
                    )
                }
            }
        }
    }
}