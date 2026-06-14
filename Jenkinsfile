pipeline {
    agent any

    environment {
        IMAGE_NAME = "007maha/python_flask"
        IMAGE_TAG  = "${BUILD_NUMBER}"  // unique tag per build
    }

    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        stage("Build Docker image") {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerhub_pat",
                    usernameVariable: "DOCKER_USER",
                    passwordVariable: "DOCKER_PASS"
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage("Update image tag in repo") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "github_pat",
                    usernameVariable: "GIT_USER",
                    passwordVariable: "GIT_PASS"
                )]) {
                    sh """
                        sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml
                        git config user.email "jenkins@ci.com"
                        git config user.name "Jenkins"
                        git add k8s/deployment.yaml
                        git commit -m "ci: update image tag to ${IMAGE_TAG}"
                        git push https://${GIT_USER}:${GIT_PASS}@github.com/007mahapra/flask_jenkins_argocd.git HEAD:main
                    """
                }
            }
        }
    }
}