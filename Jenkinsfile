pipeline {

  agent any

  stages {
    stage('Build') {
      steps {
        sh 'docker build -t my-flask-app .'
        sh 'docker tag my-flask-app $DOCKER_BFLASK_IMAGE'
      }
    }
    stage('Test') {
      steps {
        sh 'docker run my-flask-app python -m pytest app/tests/'
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
          sh 'docker push $DOCKER_BFLASK_IMAGE'
        }
      }
    }
  

    stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Retrieve the secret text and write it to a temporary file
                    withCredentials([string(credentialsId: 'my-kubeconfig-secret-id', variable: 'KUBECONFIG_CONTENT')]) {
                        writeFile file: 'temp_kubeconfig', text: env.KUBECONFIG_CONTENT
                        sh 'kubectl --kubeconfig=temp_kubeconfig apply -f deploy/deployment.yaml'
                    }
                }
            }
        }

  }
   post {
    always {
      sh 'docker logout'
    }
   }
}
