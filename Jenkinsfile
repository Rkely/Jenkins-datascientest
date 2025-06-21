pipeline {
  environment {
    DOCKER_ID = "1212455"
    MOVIE_IMAGE = "movie-service"
    CAST_IMAGE = "cast-service"
    TAG = "v.${BUILD_ID}.0"
  }
  agent any

  stages {
    stage('Build & Push Movie Service') {
         environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

      steps {
        script {
          sh '''
          docker build -t $DOCKER_ID/$MOVIE_IMAGE:$TAG ./movie-service
          echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
          docker push $DOCKER_ID/$MOVIE_IMAGE:$TAG
          '''
        }
      }
    }

    stage('Build & Push Cast Service') {
         environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }
      steps {
        script {
          sh '''
          docker build -t $DOCKER_ID/$CAST_IMAGE:$TAG ./cast-service
          echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
          docker push $DOCKER_ID/$CAST_IMAGE:$TAG
          '''
        }
      }
    }

    stage('Deploy Dev') {
      environment {
        KUBECONFIG = credentials("kubeconfig")
      }
      steps {
        script {
          sh '''
          mkdir -p ~/.kube
          cp $KUBECONFIG ~/.kube/config

          cp helm-microservices-chart/values.yaml values.yaml
          sed -i "s/tag: .*/tag: ${TAG}/" values.yaml
          helm upgrade --install microservices-app helm-microservices-chart --values values.yaml --namespace dev --create-namespace
          '''
        }
      }
    }

    stage('Deploy QA') {
      environment {
        KUBECONFIG = credentials("kubeconfig")
      }
      steps {
        script {
          sh '''
          cp $KUBECONFIG ~/.kube/config
          sed -i "s/tag: .*/tag: ${TAG}/" values.yaml
          helm upgrade --install microservices-app helm-microservices-chart --values values.yaml --namespace qa --create-namespace
          '''
        }
      }
    }

    stage('Deploy Staging') {
      environment {
        KUBECONFIG = credentials("kubeconfig")
      }
      steps {
        script {
          sh '''
          cp $KUBECONFIG ~/.kube/config
          sed -i "s/tag: .*/tag: ${TAG}/" values.yaml
          helm upgrade --install microservices-app helm-microservices-chart --values values.yaml --namespace staging --create-namespace
          '''
        }
      }
    }

    stage('Deploy Production') {
      environment {
        KUBECONFIG = credentials("kubeconfig")
      }
      steps {
        timeout(time: 15, unit: 'MINUTES') {
          input message: 'Deploy to production?', ok: 'Yes'
        }

        script {
          sh '''
          cp $KUBECONFIG ~/.kube/config
          sed -i "s/tag: .*/tag: ${TAG}/" values.yaml
          helm upgrade --install microservices-app helm-microservices-chart --values values.yaml --namespace prod --create-namespace
          '''
        }
      }
    }
  }
}
