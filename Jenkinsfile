pipeline {
  environment {
    DOCKER_ID = "1212455"
    MOVIE_IMAGE = "movie-service"
    CAST_IMAGE = "cast-service"
    TAG = "latest"
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
        KUBECONFIG = credentials("config")
      }
     steps {                 // sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
         script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp helm-microservices-chart/values.yaml values.yml
                cat values.yml
                helm upgrade --install app  helm-microservices-chart --values=values.yml --namespace dev --create-namespace
                '''
            }
      }
    }

    stage('Deploy QA') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
          script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp helm-microservices-chart/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app  helm-microservices-chart --values=values.yml --namespace qa --create-namespace
                '''
            }
      }
    }

    stage('Deploy Staging') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
         script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp helm-microservices-chart/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app  helm-microservices-chart --values=values.yml --namespace staging --create-namespace
                '''
            }
      }
    }

    stage('Deploy Production') {
      environment {
        KUBECONFIG = credentials("config")
      }
      steps {
        timeout(time: 15, unit: 'MINUTES') {
          input message: 'Deploy to production?', ok: 'Yes'
        }

        script {
           sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp helm-microservices-chart/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app  helm-microservices-chart --values=values.yml --namespace prod --create-namespace
                '''
        }
      }
    }
  }
}
