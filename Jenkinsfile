pipeline {
    agent any
    
    // 這裡定義全域環境變數，方便後續 ChatOps 使用
    environment {
        MY_NAME = "曾煥軒"
        STUDENT_ID = "B12705002"
        DISCORD_WEBHOOK = "https://discord.com/api/webhooks/1452002798479610049/4QzrPJPQ2DEvT-wg1Cjg66CZ76vE5xjlc5xs2ukvUEroFGW9H2Oo3K9Tb2JBsizoaXZl"
        REPO_NAME = "lsap-cicd-example-app"
    }

    stages {
        stage('Static Analysis') {
            tools {
                nodejs 'nodejs'
            }
            steps {
                // 此階段必須在所有分支執行 
                echo 'Running ESLint...'
                sh 'npm install'
                sh 'npm run lint'
            }
        }

        stage('Staging Deployment') {
            when { branch 'dev' }
            steps {
                // 必須使用 script 區塊才能定義變數與執行複雜邏輯
                script {
                    def imageTag = "dev-${env.BUILD_NUMBER}"
                    def fullImageName = "${env.DOCKER_HUB_USER}/${env.REPO_NAME}:${imageTag}"
                    
                    // 使用 withCredentials 綁定帳號密碼至環境變數
                    withCredentials([usernamePassword(credentialsId: 'ae58b061-6e5f-4b57-a94e-422fd6da1699', 
                                                    passwordVariable: 'DOCKER_PASSWORD', 
                                                    usernameVariable: 'DOCKER_USERNAME')]) {
                        
                        // 執行 Docker Build 與 Push
                        sh "docker build -t ${fullImageName} ."
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                        sh "docker push ${fullImageName}"
                    }

                    // Cleanup 與 Deploy
                    sh "docker rm -f dev-app || true"
                    sh "docker run -d --name dev-app -p 8081:3000 ${fullImageName}"

                    // 健康檢查驗證
                    sh "sleep 5 && curl -f http://localhost:8081/health"
                }
            }
        }
    }

    post {
        failure {
            // 當 Pipeline 失敗時，發送 ChatOps 通知
            script {
                sh """
                curl -H "Content-Type: application/json" \
                -X POST \
                -d '{
                    "content": "❌ **Build Failed!**\\n**Name:** ${env.MY_NAME}\\n**ID:** ${env.STUDENT_ID}\\n**Job:** ${env.JOB_NAME}\\n**Build:** # ${env.BUILD_NUMBER}\\n**Repo:** ${env.GIT_URL}\\n**Branch:** ${env.BRANCH_NAME}\\n**Status:** ${currentBuild.currentResult}"
                }' \
                ${DISCORD_WEBHOOK}
                """
            }
        }
    }
}