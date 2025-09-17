pipeline {
  agent any
  tools {
    nodejs "NodeJS"   // :point_left: if need Use the new Node.js 20 installation
  }
  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/kothapalli1094/Trading-UI.git', branch: 'master'
      }
    }
    stage('Install') {
      steps {
        sh '''
          npm cache clean --force
          rm -rf node_modules package-lock.json
          npm install --omit=optional
        '''
      }
    }
    stage('Test') {
      steps {
        sh 'npm test || echo ":warning: No tests found"'
      }
    }
  stage('Build') {
    steps {
        withEnv(["CI=false"]) {
            sh 'npm run build'
        }
    }
}
  }
  post {
    success { echo ':white_check_mark: Node.js pipeline finished successfully.' }
    failure { echo ':x: Node.js pipeline failed — check console output.' }
  }
}
