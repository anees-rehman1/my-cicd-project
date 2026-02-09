pipeline {
    agent any

    environment {
        DOCKERHUB = "shadow1234090"
        IMAGE = "my-cicd-project"
        BRANCH = "main"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${BRANCH}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'CleanBeforeCheckout']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/anees-rehman1/my-cicd-project.git',
                        credentialsId: 'github-credentials' // replace with your GitHub credentials ID in Jenkins
                    ]]
                ])
            }
        }

        stage('Check for Changes') {
            steps {
                script {
                    // Check if any files changed in the last commit
                    def changes = sh(script: "git diff --name-only HEAD~1 HEAD", returnStdout: true).trim()
                    if (changes == "") {
                        echo "No code changes detected. Skipping build."
                        currentBuild.result = 'SUCCESS'
                        return
                    } else {
                        echo "Code changes detected:\n${changes}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            when {
                expression { currentBuild.result != 'SUCCESS' } // only run if changes exist
            }
            steps {
                sh "docker build -t ${DOCKERHUB}/${IMAGE}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            when {
                expression { currentBuild.result != 'SUCCESS' }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push ${DOCKERHUB}/${IMAGE}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Update Deployment & Push') {
            when {
                expression { currentBuild.result != 'SUCCESS' }
            }
            steps {
                sh """
                git checkout ${BRANCH}

                # Update deployment.yaml with new image tag
                sed -i "s|image:.*|image: ${DOCKERHUB}/${IMAGE}:${BUILD_NUMBER}|g" k8s/deployment.yaml

                # Git config
                git config user.email "jenkins@local"
                git config user.name "jenkins"

                git add k8s/deployment.yaml
                git commit -m "Update deployment image to build ${BUILD_NUMBER}" || echo "No changes to commit"

                # Push using credentials
                git remote set-url origin https://USERNAME:${GITHUB_TOKEN}@github.com/anees-rehman1/my-cicd-project.git
                git push origin ${BRANCH} || echo "Push skipped if no changes"
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
