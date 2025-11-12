pipeline {
    agent {
        docker {
            image 'node:20-alpine'      // Lightweight Node.js image
            args '-u root'              // Run as root for npm install permissions
        }
    }

    environment {
        // Git repo details
        GIT_URL = 'https://github.com/kothapalli1094/Trading-UI.git'
        GIT_BRANCH = 'master'

        // React build configuration
        BUILD_DIR = 'build'
        NODE_OPTIONS = '--openssl-legacy-provider'
        CI = 'false'

        // Docker / Nginx configuration
        APP_NAME = 'trading-ui'
        CONTAINER_NAME = 'trading-ui-nginx'
        NGINX_IMAGE = 'nginx:latest'
        PORT = '8082'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📥 Cloning repository..."
                git branch: "${GIT_BRANCH}", url: "${GIT_URL}"
                echo "✅ Repository cloned successfully!"
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "📦 Installing project dependencies..."
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                echo "🧪 Running lint and basic tests (non-blocking)..."
                sh '''
                    npm run lint || echo "⚠️ Lint warnings detected (ignored)"
                    npm test || echo "⚠️ Test failures detected (ignored)"
                '''
            }
        }

        stage('Build React App') {
            steps {
                echo "🏗️ Building Trading-UI React app..."
                sh '''
                    export NODE_OPTIONS=--openssl-legacy-provider
                    export CI=false
                    npm run build
                '''
                echo "✅ React app built successfully!"
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo "📦 Archiving the build directory..."
                archiveArtifacts artifacts: "${BUILD_DIR}/**", fingerprint: true
            }
        }

        stage('Deploy to Nginx Container') {
            steps {
                echo "🚀 Deploying Trading-UI to Nginx..."
                sh '''
                    echo "🧹 Cleaning up old Nginx container if it exists..."
                    docker rm -f ${CONTAINER_NAME} || true

                    echo "🆕 Starting a fresh Nginx container..."
                    docker run -d --name ${CONTAINER_NAME} -p ${PORT}:80 ${NGINX_IMAGE}

                    echo "📁 Copying React build files to Nginx container..."
                    docker cp ${BUILD_DIR}/. ${CONTAINER_NAME}:/usr/share/nginx/html/

                    echo "✅ Deployment successful! Visit: http://$(hostname -I | awk '{print $1}'):${PORT}"
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo "🔍 Running post-deployment health check..."
                script {
                    def response = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${PORT} || true",
                        returnStdout: true
                    ).trim()
                    if (response == '200') {
                        echo "✅ Health check passed — Trading-UI is live!"
                    } else {
                        error "❌ Health check failed (Response code: ${response})"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Trading-UI pipeline executed successfully!"
            echo "🌐 Access the app at: http://$(hostname -I | awk '{print $1}'):${PORT}"
        }
        failure {
            echo "❌ Build or deployment failed. Please check the Jenkins logs above."
        }
    }
}
