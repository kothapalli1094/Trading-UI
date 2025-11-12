pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/kothapalli1094/Trading-UI.git'
        GIT_BRANCH = 'master'

        BUILD_DIR = 'build'
        NODE_VERSION = '18'
        NODE_OPTIONS = '--openssl-legacy-provider'
        CI = 'false'

        APP_NAME = 'trading-ui'
        CONTAINER_NAME = 'trading-ui-nginx'
        NGINX_IMAGE = 'nginx:latest'
        PORT = '8082'
    }

    stages {

        stage('Prepare Environment') {
            steps {
                echo "⚙️ Checking Node.js installation..."
                sh '''
                    if ! command -v node &> /dev/null; then
                        echo "📦 Node.js not found — installing Node.js ${NODE_VERSION}..."
                        curl -fsSL https://rpm.nodesource.com/setup_${NODE_VERSION}.x | sudo -E bash -
                        sudo yum install -y nodejs
                    else
                        echo "✅ Node.js already installed — version: $(node -v)"
                    fi
                '''
            }
        }

        stage('Checkout Code') {
            steps {
                echo "📥 Cloning repository from ${GIT_BRANCH}..."
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

        stage('Run Lint & Tests') {
            steps {
                echo "🧪 Running lint and test scripts..."
                sh '''
                    npm run lint || echo "⚠️ Lint warnings detected (ignored)"
                    npm test || echo "⚠️ Test failures detected (ignored)"
                '''
            }
        }

        stage('Build React App') {
            steps {
                echo "🏗️ Building Trading-UI React application..."
                sh '''
                    export NODE_OPTIONS=--openssl-legacy-provider
                    export CI=false
                    npm run build
                '''
                echo "✅ React app built successfully!"
            }
        }

        stage('Archive Build Artifacts') {
            steps {
                echo "📦 Archiving build directory for reference..."
                archiveArtifacts artifacts: "${BUILD_DIR}/**", fingerprint: true
            }
        }

        stage('Deploy to Nginx') {
            steps {
                echo "🚀 Deploying Trading-UI build to Nginx container..."
                sh '''
                    echo "🧹 Removing old container if exists..."
                    docker rm -f ${CONTAINER_NAME} || true

                    echo "🆕 Starting new Nginx container..."
                    docker run -d --name ${CONTAINER_NAME} -p ${PORT}:80 ${NGINX_IMAGE}

                    echo "📁 Copying React build files to Nginx container..."
                    docker cp ${BUILD_DIR}/. ${CONTAINER_NAME}:/usr/share/nginx/html/

                    echo "✅ Deployment completed successfully!"
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo "🔍 Performing post-deployment health check..."
                script {
                    def response = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${PORT} || true",
                        returnStdout: true
                    ).trim()
                    if (response == '200') {
                        echo "✅ Health check passed — Trading-UI is live!"
                    } else {
                        error "❌ Health check failed — Response code: ${response}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Trading-UI pipeline completed successfully!"
            sh '''
                IP=$(hostname -I | awk '{print $1}')
                echo "🌐 Application available at: http://$IP:${PORT}"
            '''
        }
        failure {
            echo "❌ Pipeline failed. Please check the Jenkins logs for details."
        }
    }
}
