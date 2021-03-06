
pipeline {
  agent any
  options {
    skipDefaultCheckout(true)
  }
  environment {
    imageName = 'crsanti/my-api-app:latest'
    ec2Instance = 'ec2-54-170-39-116.eu-west-1.compute.amazonaws.com'
    appPort = 80
    githubAccount = "crsanti"
    githubRepoName = "my-api-app"
  }
  stages {
    stage('Warmup') {
      steps {
        sh 'pwd; ls -l'
        cleanWs()
        sh 'pwd; ls -l'
        checkout scm
      }
    }
    stage('Notify GitHub build in progress') {
      steps {
        githubNotify(
          status: "PENDING",
          credentialsId: "github-commit-status-credentials",
          account: githubAccount,
          repo: githubRepoName,
          description: "Some checks haven't completed yet"
        )
      }
    }
    stage('Install dependencies') {
      agent {
        docker {
          image 'node:14-alpine'
          reuseNode true
        }
      }
      steps {
        sh 'npm ci'
      }
    }
    stage('Tests') {
      agent {
        docker {
          image 'node:14-alpine'
          reuseNode true
        }
      }
      steps {
        sh 'npm test'
      }
    }
    stage('E2E Tests') {
      when {
        branch 'staging'
      }
      agent {
        docker {
          image 'node:14-alpine'
          reuseNode true
        }
      }
      environment {
        BASE_API_URL = "http://$ec2Instance:$appPort"
      }
      steps {
        sh 'npm run test:e2e'
      }
    }
    stage('Build image & push it to DockerHub') {
      when {
        branch 'develop'
      }
      agent {
        docker {
          image 'node:14-alpine'
          reuseNode true
        }
      }
      steps {
        script {
          def dockerImage = docker.build(imageName)
          withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
            dockerImage.push()
            sh 'docker rmi $imageName'
          }
        }
      }
    }
    stage('Deploy to server') {
      when {
        branch 'develop'
      }
      environment {
        containerName = 'my-api-app'
      }
      steps {
        withCredentials([
          sshUserPrivateKey(
            credentialsId: 'ec2-ssh-credentials',
            keyFileVariable: 'identityFile',
            passphraseVariable: 'passphrase',
            usernameVariable: 'user'
          )
        ]) {
          sh '''
            ssh -o StrictHostKeyChecking=no -i $identityFile $user@$ec2Instance \
            APP_PORT=$appPort CONTAINER_NAME=$containerName IMAGE_NAME=$imageName bash < ./scripts/deploy.sh
          '''
        }
      }
    }
  }
  post {
    success {
      githubNotify(
        status: "SUCCESS",
        credentialsId: "github-commit-status-credentials",
        account: githubAccount,
        repo: githubRepoName,
        description: "All checks have passed"
      )
    }
    failure {
      githubNotify(
        status: "FAILURE",
        credentialsId: "github-commit-status-credentials",
        account: githubAccount,
        repo: githubRepoName,
        description: "Some checks were not successful"
      )
    }
  }
}
