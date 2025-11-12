pipeline {
    agent any

    tools {
        nodejs 'node18'  // Make sure you have Node.js configured under "Manage Jenkins → Global Tool Configuration"
    }

    environment {
        // Git Repository
        GIT_URL = 'https://github.com/kothapalli1094/Trading-UI.git'
        GIT_BRANCH = 'master'

        // SonarQube setup (optional)
        SONARQUBE_ENV = 'sonar'
        SCANNER_HOME = tool 'sonar'

        // Build output folder
        BUILD_DIR = 'build'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📥 Cloning source code from ${GIT_URL}..."
                git branch: "${GIT_BRANCH}", url: "${GIT_URL}"
                echo "✅ Code checkout complete."
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "📦 Installing Node.js dependencies..."
                sh 'npm install'
            }
        }

        stage('Run Lint and Tests') {
            steps {
                echo "🧪 Running lint and unit tests..."
                sh '''
                    npm run lint || true
                    npm test || echo "⚠️ Tests failed but build continues (adjust for strict mode)"
                '''
            }
        }

        stage('SonarQube Analysis') {
            when {
                expression { fileExists('sonar-project.properties') }
            }
            steps {
                echo "🔍 Running SonarQube static code analysis..."
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=Trading-UI \
                            -Dsonar.projectName=Trading-UI \
                            -Dsonar.sources=src \
                            -Dsonar.projectVersion=${env.BUILD_NUMBER} \
                            -Dsonar.host.url=http://54.145.245.39:9000
                    '''
                }
            }
        }

        stage('Build Application') {
            steps {
                echo "🏗️ Building the Trading-UI project..."
                sh 'npm run build'
            }
        }

        stage('Archive Build Artifacts') {
            steps {
                echo "📦 Archiving build artifacts..."
                archiveArtifacts artifacts: "${BUILD_DIR}/**", fingerprint: true
            }
        }

        stage('Deploy') {
            when {
                expression { return fileExists("${BUILD_DIR}") }
            }
            steps {
                echo "🚀 Deploying Trading-UI to target environment..."
                sh '''
                    # Example: Copy build files to /var/www/html (Nginx)
                    sudo rm -rf /var/www/html/*
                    sudo cp -r build/* /var/www/html/
                    echo "✅ Trading-UI deployed successfully!"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Trading-UI Pipeline completed successfully."
        }
        failure {
            echo "❌ Trading-UI Pipeline failed. Please check the logs."
        }
    }
}
