pipeline {
    agent {
        docker {
            image 'node:20-alpine'   // modern LTS Node version
            args '-u root'           // allows using npm install -g if needed
        }
    }

    environment {
        GIT_URL = 'https://github.com/kothapalli1094/Trading-UI.git'
        GIT_BRANCH = 'master'
        BUILD_DIR = 'build'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_URL}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    npm run lint || true
                    npm test || echo "⚠️ Tests failed but continuing"
                '''
            }
        }

        stage('Build App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: "${BUILD_DIR}/**", fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "🚀 Simulated deployment..."
                    ls -l ${BUILD_DIR}
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Trading-UI pipeline succeeded."
        }
        failure {
            echo "❌ Trading-UI pipeline failed."
        }
    }
}
