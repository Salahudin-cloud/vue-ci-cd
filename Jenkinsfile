pipeline {
  agent any

  tools {
    nodejs 'node23'
  }

  environment {
    APP_NAME = "vue-app"
    PORT = "5173"
  }

  stages {
    stage('Clone') {
      steps {
        sh 'git config --global http.postBuffer 524288000'
        sh 'rm -rf vue-ci-cd && git clone --depth=1 https://github.com/Salahudin-cloud/vue-ci-cd.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        dir('vue-ci-cd') {
          sh 'npm install -g yarn'
          sh 'yarn install'
        }
      }
    }

    stage('Build') {
      steps {
        dir('vue-ci-cd') {
          sh 'yarn build'
        }
      }
    }

    stage('Deploy with Nginx') {
      steps {
        dir('vue-ci-cd') {
          sh 'sudo rm -rf /var/www/html/*'
          sh 'sudo cp -r dist/* /var/www/html/'
        }
      }
    }

    stage('Serve with PM2 (optional)') {
      steps {
        dir('vue-ci-cd') {
          sh 'npm install -g pm2'
          sh 'pm2 delete ${APP_NAME} || true'
          sh 'pm2 serve dist ${PORT} --name ${APP_NAME} --spa'
          sh 'pm2 save'
        }
      }
    }
  }
}
