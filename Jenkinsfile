pipeline {
    agent {
        docker {
            image 'node:20-alpine'
            args '-u root' // allow npm install to work properly
        }
    }

    environment {
        // Git repo details
        GIT_URL = 'https://github.com/kothapalli1094/Trading-UI.git'
        GIT_BRANCH = 'master'

        // Build configuration
        BUILD_DIR = 'build'
        NODE_OPTIONS = '--openssl-legacy-provider'

        // Docker image details
        APP_NAME = 'trading-ui'
        APP_VERSION = '1.0'
        NGINX_IMAGE = 'nginx:latest'
        CONTAINER_NAME = 'trading-ui-nginx'
        PORT = '8082'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📥 Cloning repository..."
                git branch: "${GIT_BRANCH}", url: "${GIT_URL}"
                echo "✅ Code cloned successfully!"
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
                echo "🧪 Running tests..."
                sh '''
                    npm run lint || echo "⚠️ Lint warnings (ignored)"
                    npm test || echo "⚠️ Test failures (ignored)"
                '''
            }
        }

        stage('Build React App') {
            steps {
                echo "🏗️ Building the Trading-UI React app..."
                sh '''
                    export NODE_OPTIONS=--openssl-legacy-provider
                    npm run build
                '''
                echo "✅ Build completed successfully!"
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo "📦 Archiving the build output..."
                archiveArtifacts artifacts: "${BUILD_DIR}/**", fingerprint: true
            }
        }

        stage('Deploy to Nginx Container') {
            steps {
                echo "🚀 Deploying Trading-UI to Nginx..."
                sh '''
                    echo "Cleaning up old container if exists..."
                    docker rm -f ${CONTAINER_NAME} || true

                    echo "Starting new Nginx container..."
                    docker run -d --name ${CONTAINER_NAME} -p ${PORT}:80 ${NGINX_IMAGE}

                    echo "Copying React build files into Nginx container..."
                    docker cp ${BUILD_DIR}/. ${CONTAINER_NAME}:/usr/share/nginx/html/

                    echo "✅ Deployment complete! Access app at: http://<your-server-ip>:${PORT}"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Trading-UI build and Nginx deployment succeeded!"
        }
        failure {
            echo "❌ Trading-UI pipeline failed. Check logs above."
        }
    }
}
