pipeline {
    agent any
    
    tools {
        jdk 'jdk'
        maven 'maven'
    }
    
    parameters {
        string(name: 'GITHUB_REPO',
               defaultValue: 'https://github.com/vwork900/BankingUnitTests.git',
               description: 'GitHub repository URL')
        string(name: 'BRANCH',
               defaultValue: 'master',
               description: 'Branch name')
        string(name: 'RP_ENDPOINT',
               defaultValue: 'http://localhost:8080',
               description: 'ReportPortal URL')
        string(name: 'RP_PROJECT',
               defaultValue: 'superadmin_personal',
               description: 'ReportPortal project')
        string(name: 'LAUNCH_NAME',
               defaultValue: '',
               description: 'Custom launch name (optional, defaults to job name)')
    }
    
    environment {
        RP_UUID = credentials('reportportal-uuid')
        LAUNCH_NAME = "${params.LAUNCH_NAME ?: env.JOB_NAME}_Build_${env.BUILD_NUMBER}"
    }
    
    stages {
        // =========================
        // 1. Checkout Code
        // =========================
        stage('Checkout') {
            steps {
                echo "Cloning repository: ${params.GITHUB_REPO}"
                echo "Branch: ${params.BRANCH}"
                git branch: "${params.BRANCH}",
                    url: "${params.GITHUB_REPO}",
                    credentialsId: 'github-credentials'
            }
        }
        
        // =========================
        // 2. Build Project
        // =========================
        stage('Build') {
            steps {
                echo "Building project..."
                bat 'mvn clean compile'
            }
        }
        
        // =========================
        // 3. Run Unit Tests
        // =========================
        stage('Run Tests') {
            steps {
                echo "Running unit tests..."
                echo "ReportPortal Project: ${params.RP_PROJECT}"
                echo "Launch Name: ${env.LAUNCH_NAME}"
                bat """
                    mvn test ^
                    -Drp.endpoint=${params.RP_ENDPOINT} ^
                    -Drp.uuid=${env.RP_UUID} ^
                    -Drp.project=${params.RP_PROJECT} ^
                    -Drp.launch=${env.LAUNCH_NAME} ^
                    -Drp.attributes=build:${env.BUILD_NUMBER};branch:${params.BRANCH};job:${env.JOB_NAME}
                """
            }
        }
        
        // =========================
        // 4. Publish Test Reports
        // =========================
        stage('Publish Results') {
            steps {
                echo "Publishing test results..."
                script {
                    junit allowEmptyResults: true, 
                          testResults: '**/target/surefire-reports/*.xml'
                    
                    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                    echo "📊 ReportPortal Results:"
                    echo "🔗 ${params.RP_ENDPOINT}/ui/#${params.RP_PROJECT}/launches/all"
                    echo "🚀 Launch: ${env.LAUNCH_NAME}"
                    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo "Pipeline execution finished"
                echo "Build Number: ${env.BUILD_NUMBER}"
                try {
                    cleanWs()
                    echo "Workspace cleaned successfully"
                } catch (Exception e) {
                    echo "Warning: Could not clean workspace: ${e.message}"
                }
            }
        }
        success {
            echo "✅ Build & Tests Successful"
            echo "📊 View results: ${params.RP_ENDPOINT}/ui/#${params.RP_PROJECT}/launches/all"
        }
        failure {
            echo "❌ Build or Tests Failed"
            echo "🔍 Check logs above for details"
        }
    }
}
