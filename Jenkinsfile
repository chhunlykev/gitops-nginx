pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/you/app-repo'
      }
    }

    stage('Trivy Scan') {
      steps {
        sh 'trivy fs . --exit-code 1 --severity HIGH,CRITICAL || true'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t yourdockerhub/nginx-web:latest .'
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'DOCKER_PASS')]) {
          sh '''
            echo $DOCKER_PASS | docker login -u yourdockerhub --password-stdin
            docker push yourdockerhub/nginx-web:latest
          '''
        }
      }
    }

    stage('Update GitOps Repo') {
      steps {
        sh '''
        git clone https://github.com/you/gitops-repo.git
        cd gitops-repo/nginx
        sed -i 's|image: .*|image: yourdockerhub/nginx-web:latest|' deployment.yaml
        git commit -am "Update image"
        git push
        '''
      }
    }
  }
}
