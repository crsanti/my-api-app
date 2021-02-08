pipeline {
  agent any
  stages {
    stage('Install dependencies') {
      agent {
        docker {
          image 'node:14-alpine'
        }
      }
      steps {
        sh 'npm ci'
      }
    }
    stage('Build') {
      agent {
        docker {
          image 'node:14-alpine'
        }
      }
      steps {
        sh 'npm run build'
      }
    }
    stage('Tests') {
      agent {
        docker {
          image 'node:14-alpine'
        }
      }
      steps {
        sh 'npm test'
      }
    }
  }
}
