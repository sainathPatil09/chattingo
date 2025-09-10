@Library('Shared') _
pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }

    // Define Docker registry and image names
    environment {
        REGISTRY = "wiings09"
        FRONTEND_IMAGE = "chattingo-frontend"
        BACKEND_IMAGE = "chattingo-backend"
        TAG = "${env.BUILD_NUMBER}" // unique tag per build; can replace with commit hash if desired
        SONAR_HOME = tool "Sonar"
        BACKEND_DIR = "backend"
        FRONTEND_DIR = "frontend"
        
    }

    // Parameters for optional manual image tag
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Optional: specify a custom Docker image tag')
    }

    stages {
        // Stage 1: Clone Git repository
        stage('Git: Code Checkout') {
            steps {
                script{
                    code_checkout("https://github.com/sainathPatil09/chattingo.git","main")
                }
            }
        }
        
        stage("File System Scan"){
            parallel{
                stage('SonarQube Backend'){
                    steps{
                        script {
                            sonarBackend(env.BACKEND_DIR, "chattingo-backend", "Chattingo Backend")
                       }
                    }
                }
                
                stage('SonarQube Frontend'){
                    steps{
                        script {
                            sonarFrontend(env.FRONTEND_DIR, "chattingo-frontend", "Chattingo Frontend")
                        }
                    }
                }
                stage('Trivy FS'){
                    steps{
                        script {
                            trivyFsScan()
                        }
                    }
                }
            }
        }

        // Stage 2: Build Docker images
        stage('Image Build') {
            steps {
                script {
                    def finalTag = params.IMAGE_TAG ?: TAG
                    dockerBuild(
                        env.REGISTRY,
                        env.FRONTEND_IMAGE,
                        env.BACKEND_IMAGE,
                        finalTag,
                        "http://72.60.111.17.nip.io:8000", // API URL (could also come from Jenkins parameter)
                        env.FRONTEND_DIR,
                        env.BACKEND_DIR
                    )
                }
            }
        }
        
        // Image Scan
        stage('Image Scan') {
            steps {
                script {
                    // Scan backend image
                    def finalTag = params.IMAGE_TAG ?: TAG
                    dockerScan(
                        env.REGISTRY,
                        env.FRONTEND_IMAGE,
                        env.BACKEND_IMAGE,
                        finalTag
                    )
                }
            }
        }

        // Stage 5: Push Docker images to registry
        stage('Push to Registry') {
            steps {
                script {
                    def finalTag = params.IMAGE_TAG ?: TAG
                    dockerPush(
                        env.REGISTRY,
                        env.FRONTEND_IMAGE,
                        env.BACKEND_IMAGE,
                        finalTag,
                        'dockerhub-creds' // Jenkins credentials ID
                    )
                }
            }
        }

        // Stage 6: Deploy on the same VPS
        stage('Deploy') {
            steps {
                script {
                    deployCompose(
                        "/var/lib/jenkins/workspace/Chattingo-CICD",
                        env.FINAL_TAG,
                        [
                            'JWT_SECRET'          : 'jwt-secret',
                            'SPRING_DATASOURCE_URL': 'db-url',
                            'SPRING_DATASOURCE_USERNAME': 'db-user',
                            'SPRING_DATASOURCE_PASSWORD': 'db-pass',
                            'CORS_ALLOWED_ORIGINS': 'cors-origins',
                            'CORS_ALLOWED_METHODS': 'cors-methods',
                            'CORS_ALLOWED_HEADERS': 'cors-headers',
                            'SPRING_PROFILES_ACTIVE': 'spring-profile',
                            'SERVER_PORT': 'server-port',
                            'MYSQL_ROOT_PASSWORD': 'mysql-root-pass',
                            'MYSQL_DATABASE': 'mysql-db'
                        ]
                    )
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful 🚀"
        }
        failure {
            echo "❌ Deployment failed. Check logs."
        }
    }
}
