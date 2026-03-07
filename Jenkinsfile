pipeline {
  environment {
    DOCKER_ID = 'devopsdsgc'
    DOCKER_IMAGE = 'datascientestapi'
    DOCKER_TAG = "v.${BUILD_ID}.0"
  }
  agent any
  stages {
    stage(' Docker Build') {
      steps {
        script {
          sh '''
        docker rm -f jenkins
        docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
      sleep 6
      '''
        }
      }
    }
    stage('Docker run') {
      steps {
        script {
          sh '''
      docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
      sleep 10
      '''
        }
      }
    }
    stage('Test Acceptance') {
      steps {
        script {
          sh '''
      curl localhost
      '''
        }
      }
    }
    stage('Docker Push') {
      environment
    {
        DOCKER_PASS = credentials('DOCKER_HUB_PASS')
    }
      steps {
        script {
          sh '''
        docker login -u $DOCKER_ID -p $DOCKER_PASS
        docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
        '''
        }
      }
    }
    stage('Deploiement en dev') {
      environment {
        KUBECONFIG = credentials('config')
      }
      steps {
        script {
          sh '''
      rm -Rf .kube
      mkdir .kube
      ls
      cat $KUBECONFIG > .kube/config
      cp fastapi/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      helm upgrade --install app fastapi --values=values.yml --namespace dev
      '''
        }
      }
    }
    stage('Deploiement en staging') {
      environment {
        KUBECONFIG = credentials('config')
      }
      steps {
        script {
          sh '''
      rm -Rf .kube
      mkdir .kube
      ls
      cat $KUBECONFIG > .kube/config
      cp fastapi/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      hel m upgrade --install app fastapi --values=values.yml --namespace staging
      '''
        }
      }
    }
    stage('Deploiement en prod') {
      environment {
        KUBECONFIG = credentials('config') // we retrieve  kubeconfig from secret file called config saved on jenkins
      }
      steps {
        // Create an Approval Button with a timeout of 15minutes.
        // this require a manuel validation in order to deploy on production environment
        timeout(time: 15, unit: 'MINUTES') {
          input message: 'Do you want to deploy in production ?', ok: 'Yes'
        }
        script {
          sh '''
          rm -Rf .kube
          mkdir .kube
          ls
          cat $KUBECONFIG > .kube/config
          cp fastapi/values.yaml values.yml
          cat values.yml
          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
          helm upgrade --install app fastapi --values=values.yml --namespace prod
          '''
        }
      }
    }    
  }
  post { // send email when the job has failed
        failure {
            echo "This will run if the job failed :)"
            mail to: "gch4rles@gmail.com",
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
    }
}
