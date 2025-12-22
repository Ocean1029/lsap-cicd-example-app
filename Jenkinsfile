pipeline {
    agent any
    environment {
        MY_NAME = "曾煥軒"
        STUDENT_ID = "B12705002"
        DISCORD_WEBHOOK = "https://discord.com/api/webhooks/1452002798479610049/4QzrPJPQ2DEvT-wg1Cjg66CZ76vE5xjlc5xs2ukvUEroFGW9H2Oo3K9Tb2JBsizoaXZl"
        REPO_NAME = "lsap-cicd-example-app"
    }

    stages {
        stage('Static Analysis') {
            tools { nodejs 'nodejs' }
            steps {
                echo 'Running ESLint...'
                sh 'npm install'
                sh 'npm run lint'
            }
        }

        stage('Staging Deployment') {
            when { branch 'dev' }
            steps {
                script {
                    def imageTag = "dev-${env.BUILD_NUMBER}"
                    def fullImageName = "" 

                    withCredentials([usernamePassword(credentialsId: 'ae58b061-6e5f-4b57-a94e-422fd6da1699', 
                                                    passwordVariable: 'DOCKER_PASSWORD', 
                                                    usernameVariable: 'DOCKER_USERNAME')]) {
                        
                        fullImageName = "${DOCKER_USERNAME}/${env.REPO_NAME}:${imageTag}"
                        
                        sh "docker build -t ${fullImageName} ."
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                        sh "docker push ${fullImageName}"
                    }

                    sh "docker rm -f dev-app || true"
                    sh "docker run -d --name dev-app -p 8081:3000 ${fullImageName}"
                    sh "sleep 5 && curl -f http://localhost:8081/health"
                }
            }
        }

        stage('Production Promotion (GitOps)') {
            when { branch 'main' }
            steps {
                script {
                    // 1. 讀取配置：從 deploy.config 讀取目標標籤 
                    def TARGET_TAG = readFile('deploy.config').trim()
                    def prodTag = "prod-${env.BUILD_NUMBER}"
                    def promotedImage = ""

                    withCredentials([usernamePassword(credentialsId: 'ae58b061-6e5f-4b57-a94e-422fd6da1699', 
                                                    passwordVariable: 'DOCKER_PASSWORD', 
                                                    usernameVariable: 'DOCKER_USERNAME')]) {
                        
                        def sourceImage = "${DOCKER_USERNAME}/${env.REPO_NAME}:${TARGET_TAG}"
                        promotedImage = "${DOCKER_USERNAME}/${env.REPO_NAME}:${prodTag}"

                        // 2. 映像檔晉升：拉取舊標籤、重標記、推送新標籤 
                        sh "docker pull ${sourceImage}"
                        sh "docker tag ${sourceImage} ${promotedImage}"
                        sh "docker push ${promotedImage}"
                    }

                    // 3. 部署：清理舊容器並在 8082 啟動 [cite: 63, 64, 66]
                    sh "docker rm -f prod-app || true"
                    sh "docker run -d --name prod-app -p 8082:3000 ${promotedImage}"
                }
            }
        }
    }

    post {
        failure {
            script {
                sh """
                curl -H "Content-Type: application/json" \
                -X POST \
                -d '{
                    "content": "❌ **Build Failed!**\\n**Name:** ${env.MY_NAME}\\n**ID:** ${env.STUDENT_ID}\\n**Job:** ${env.JOB_NAME}\\n**Build:** # ${env.BUILD_NUMBER}\\n**Repo:** ${env.GIT_URL}\\n**Branch:** ${env.BRANCH_NAME}\\n**Status:** ${currentBuild.currentResult}"
                }' \
                ${env.DISCORD_WEBHOOK}
                """
            } 
        }
    }
}