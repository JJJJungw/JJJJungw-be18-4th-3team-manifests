pipeline {
    agent { label 'ci-agent' }

    parameters {
        string(name: 'DOCKER_IMAGE_VERSION', defaultValue: '', description: 'Docker Image Version')
        string(name: 'DID_BUILD_APP', defaultValue: '', description: 'Did Build Frontend')
        string(name: 'DID_BUILD_API', defaultValue: '', description: 'Did Build Backend')
    }

    environment {
        GIT_CREDENTIALS_ID = 'github-deploy-key'
        GIT_USER_NAME = 'JJJJungw'
        GIT_USER_EMAIL = 'dyungwoo3600@gmail.com'  // 네 깃허브 메일로 바꿔
    }

    stages {
        stage('Checkout main branch') {
            steps {
                checkout scm
                sh 'git checkout main || true'
                echo "   Received Params:"
                echo "   DOCKER_IMAGE_VERSION: ${params.DOCKER_IMAGE_VERSION}"
                echo "   DID_BUILD_APP: ${params.DID_BUILD_APP}"
                echo "   DID_BUILD_API: ${params.DID_BUILD_API}"
            }
        }

        stage('Update Frontend manifest') {
            when {
                expression { params.DID_BUILD_APP == "true" }
            }
            steps {
                dir('frontend') {
                    sh '''
                        echo "🔧 Updating frontend manifest..."
                        sed -i "s|amicitia/lumi-frontend:.*|amicitia/lumi-frontend:${DOCKER_IMAGE_VERSION}|g" frontend-deploy.yaml
                        git status
                    '''
                }
            }
        }

        stage('Update Backend manifest') {
            when {
                expression { params.DID_BUILD_API == "true" }
            }
            steps {
                dir('backend') {
                    sh '''
                        echo " Updating backend manifest..."
                        sed -i "s|amicitia/lumi-backend:.*|amicitia/lumi-backend:${DOCKER_IMAGE_VERSION}|g" backend-deploy.yaml
                        git status
                    '''
                }
            }
        }

        stage('Commit & Push Changes') {
            when {
                expression {
                    return params.DID_BUILD_APP == "true" || params.DID_BUILD_API == "true"
                }
            }
            steps {
                script {
                    echo "📤 Committing updated manifests..."
                    sh '''
                        git config user.name "${GIT_USER_NAME}"
                        git config user.email "${GIT_USER_EMAIL}"
                        git add .
                        git commit -m "chore: update image tag ${DOCKER_IMAGE_VERSION}" || echo "No changes to commit"
                    '''

                    sshagent([GIT_CREDENTIALS_ID]) {
                        sh 'git push origin main'
                    }
                }
            }
        }

        stage('Trigger ArgoCD Sync') {
            steps {
                echo " ArgoCD will auto-sync manifests after commit"
            }
        }
    }

    post {
        success {
            echo " Manifests updated successfully"
        }
        failure {
            echo " Failed to update manifests"
        }
    }
}
