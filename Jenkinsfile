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

    //  stage('Undeploy Envs (Dev, QA, Staging)') {
    //   environment {
    //     KUBECONFIG = credentials("config")
    //   }
    //   steps {

    //     script {
    //       sh '''
    //       rm -Rf .kube
    //       mkdir .kube
    //       cat $KUBECONFIG > .kube/config

    //       # Suppression des releases Helm
    //       helm uninstall app --namespace dev || echo "dev: pas de release"
    //       helm uninstall app --namespace qa || echo "qa: pas de release"
    //       helm uninstall app --namespace staging || echo "staging: pas de release"
    //       '''
    //     }
    //   }
    // }


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
                # Undeploy if exists
                cp helm-microservices-chart/values-dev.yaml values.yml
                cat values.yml
                helm upgrade --install app ./helm-microservices-chart --values=values.yml --namespace dev --create-namespace
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
                cp helm-microservices-chart/values-qa.yaml values.yml
                cat values.yml
                helm upgrade --install app ./helm-microservices-chart --values=values.yml --namespace qa --create-namespace
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
                cp helm-microservices-chart/values-staging.yaml values.yml
                cat values.yml
                helm upgrade --install app ./helm-microservices-chart --values=values.yml --namespace staging --create-namespace
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
            cat $KUBECONFIG > .kube/config

            helm uninstall app --namespace prod || true

            cp helm-microservices-chart/values-prod.yaml values.yml
            cat values.yml

            helm upgrade --install app ./helm-microservices-chart --values=values.yml --namespace prod --create-namespace
          '''
        }
      }
    }
  }
}
