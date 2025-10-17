pipeline {
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: git
    image: alpine/git:2.45.2
    command: ['cat']
    tty: true
  - name: docker
    image: docker:28.5.1-cli-alpine3.22
    command: ['cat']
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
        }
    }

    parameters {
        string(name: 'DOCKER_IMAGE_VERSION', defaultValue: '', description: 'Docker Image Version')
        string(name: 'DID_BUILD_APP', defaultValue: '', description: 'Did Build APP')
        string(name: 'DID_BUILD_API', defaultValue: '', description: 'Did Build API')
    }

    environment {
        GIT_CREDENTIALS_ID = 'git_deploy'
        GIT_USER_NAME = 'JJJJungw'
        GIT_USER_EMAIL = 'dyungwoo3600@gmail.com'
        GIT_REPO_URL = 'git@github.com:JJJJungw/JJJJungw-be18-4th-3team-manifests.git'
    }

    stages {
        stage('Checkout main branch') {
            steps {
                container('git') {
                    sshagent([GIT_CREDENTIALS_ID]) {
                        sh '''
                            git config --global --add safe.directory /home/jenkins/agent/workspace/lumi-manifests
                            git remote set-url origin $GIT_REPO_URL

                            # ✅ SSH 디렉토리 및 known_hosts 등록
                            mkdir -p ~/.ssh
                            chmod 700 ~/.ssh
                            ssh-keyscan github.com >> ~/.ssh/known_hosts
                            chmod 644 ~/.ssh/known_hosts

                            # ✅ 최신 main 브랜치 가져오기
                            git fetch origin main
                            git checkout main
                            git pull origin main
                        '''
                    }
                    echo "📦 Checked out main branch"
                    echo "DOCKER_IMAGE_VERSION: ${params.DOCKER_IMAGE_VERSION}"
                    echo "DID_BUILD_APP: ${params.DID_BUILD_APP}"
                    echo "DID_BUILD_API: ${params.DID_BUILD_API}"
                }
            }
        }

        stage('Update Frontend manifest') {
            when { expression { params.DID_BUILD_APP == "true" } }
            steps {
                container('git') {
                    dir('frontend') {
                        sh '''
                            echo "🔧 Updating frontend manifest..."
                            sed -i "s|amicitia/lumi-frontend:.*|amicitia/lumi-frontend:$DOCKER_IMAGE_VERSION|g" frontend-deploy.yaml
                            git status
                        '''
                    }
                }
            }
        }

        stage('Update Backend manifest') {
            when { expression { params.DID_BUILD_API == "true" } }
            steps {
                container('git') {
                    dir('backend') {
                        sh '''
                            echo "🔧 Updating backend manifest..."
                            sed -i "s|amicitia/lumi-backend:.*|amicitia/lumi-backend:$DOCKER_IMAGE_VERSION|g" backend-deploy.yaml
                            git status
                        '''
                    }
                }
            }
        }

        stage('Commit & Push') {
            when { expression { params.DID_BUILD_API == "true" || params.DID_BUILD_APP == "true" } }
            steps {
                container('git') {
                    sshagent([GIT_CREDENTIALS_ID]) {
                        sh '''
                            git config user.name "$GIT_USER_NAME"
                            git config user.email "$GIT_USER_EMAIL"

                            git add .
                            git commit -m "chore: update image tag $DOCKER_IMAGE_VERSION" || echo "No changes to commit"

                            # ✅ SSH 재등록 (Pod은 매번 새로 뜨니까)
                            mkdir -p ~/.ssh
                            chmod 700 ~/.ssh
                            ssh-keyscan github.com >> ~/.ssh/known_hosts
                            chmod 644 ~/.ssh/known_hosts

                            git push origin main
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Manifests updated successfully and pushed to GitHub"
        }
        failure {
            echo "❌ Failed to update manifests"
        }
    }
}
