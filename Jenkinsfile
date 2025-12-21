pipeline {
    agent any
    
    // 這裡定義全域環境變數，方便後續 ChatOps 使用
    environment {
        MY_NAME = "曾煥軒"
        STUDENT_ID = "B12705002"
        DISCORD_WEBHOOK = "https://discord.com/api/webhooks/1452002798479610049/4QzrPJPQ2DEvT-wg1Cjg66CZ76vE5xjlc5xs2ukvUEroFGW9H2Oo3K9Tb2JBsizoaXZl"
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