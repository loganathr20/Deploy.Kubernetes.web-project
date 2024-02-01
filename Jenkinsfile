pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "your-docker-registry"
//test info


        // test
        K8S_NAMESPACE = "your-kubernetes-namespace"
        NEXUS_RELEASE_REPO = "your-nexus-repo/releases"
        NEXUS_SNAPSHOT_REPO = "your-nexus-repo/snapshots"
        NEXUS_USERNAME = credentials('your-nexus-credentials-id').username
        NEXUS_PASSWORD = credentials('your-nexus-credentials-id').password
    }

    stages {
        stage('Checkout') {
            steps {
                // scm
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    def mvnHome = tool 'Maven'
                    sh "${mvnHome}/bin/mvn clean package -DskipTests"
                }
            }
        }

        stage('Dockerize') {
            steps {
                script {
                    def dockerImage = "${DOCKER_REGISTRY}/${env.JOB_NAME}:${env.BUILD_NUMBER}"
                    def mvnHome = tool 'Maven'
                    sh "${mvnHome}/bin/mvn docker:build -Ddocker.image=${dockerImage}"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    def mvnHome = tool 'Maven'
                    sh "${mvnHome}/bin/mvn test"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def kubeconfig = readFile('path/to/your/kubeconfig')
                    withKubeConfig(credentialsId: 'your-kubeconfig-credentials-id', kubeconfig: kubeconfig) {
                        def dockerImage = "${DOCKER_REGISTRY}/${env.JOB_NAME}:${env.BUILD_NUMBER}"
                        sh "kubectl --namespace=${K8S_NAMESPACE} set image deployment/${env.JOB_NAME} ${env.JOB_NAME}=${dockerImage}"
                    }
                }
            }
        }

        stage('Store Artifacts in Nexus') {
            steps {
                script {
                    def mvnHome = tool 'Maven'
                    def nexusRepo = env.BRANCH_NAME.startsWith('master') ? NEXUS_RELEASE_REPO : NEXUS_SNAPSHOT_REPO
                    sh "${mvnHome}/bin/mvn deploy -DaltDeploymentRepository=${nexusRepo}::default::https://${NEXUS_USERNAME}:${NEXUS_PASSWORD}@your-nexus-repo/repository/maven-snapshots/"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded. Perform additional actions if needed.'
        }
        failure {
            echo 'Pipeline failed. Handle errors appropriately.'
        }
    }
}

