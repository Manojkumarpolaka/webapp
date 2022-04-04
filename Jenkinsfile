pipeline {
  agent node

    environment {
        DOCKER_CREDENTIAL_ID = 'dockerhub-id'
        REGISTRY = 'docker.io'
        DOCKERHUB_NAMESPACE = 'docker_username'
        APP_NAME = 'devops-java-sample'
    }

    stages {
        stage ('checkout scm') {
            agent { label 'jenkins-master' }
            steps {
                checkout(scm)
            }
        }

        stage ('build & push') {
        agent { label 'ansible-ctl-node' }
            steps {
                sh 'mvn  clean package'
                sh 'ansible-playbook -i hosts dockerimage.yml .'
                withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID"]) {
                sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
                sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER'
                
            }
        }

        stage('deploy to dev') {
          when{
            branch 'master'
          }
          steps {
            input(id: 'deploy-to-dev', message: 'deploy to dev?')
            kubernetesDeploy(configs: 'deploy/dev-ol/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
          }
        }
        stage('push with tag'){
          when{
            expression{
              return params.TAG_NAME =~ /v.*/
            }
          }
          steps {
              container ('maven') {
                input(id: 'release-image-with-tag', message: 'release image with tag?')
                  withCredentials([usernamePassword(credentialsId: "$GITHUB_CREDENTIAL_ID", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh 'git config --global user.email "kubesphere@yunify.com" '
                    sh 'git config --global user.name "kubesphere" '
                    sh 'git tag -a $TAG_NAME -m "$TAG_NAME" '
                    sh 'git push http://$GIT_USERNAME:$GIT_PASSWORD@github.com/$GITHUB_ACCOUNT/devops-java-sample.git --tags --ipv4'
                  }
                sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME '
                sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME '
          }
          }
        }
        stage('deploy to production') {
          when{
            expression{
              return params.TAG_NAME =~ /v.*/
            }
          }
          steps {
            input(id: 'deploy-to-production', message: 'deploy to production?')
            kubernetesDeploy(configs: 'deploy/prod-ol/**', enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
          }
        }
    }
}