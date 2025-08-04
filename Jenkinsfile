pipeline {
  agent any

  tools {
    nodejs 'node23'
  }

  stages {
    stage('Clone') {
      steps {
        git 'https://github.com/Salahudin-cloud/vue-ci-cd.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm install -g yarn'
        sh 'yarn install'
      }
    }

    stage('Build') {
      steps {
        sh 'yarn build'
      }
    }

    stage('Deploy Locally') {
      steps {
        sh 'fuser -k 5173/tcp || true'
        sh 'npm install -g serve'
        sh 'serve -s dist -l 5173 &'
      }
    }
  }
}
