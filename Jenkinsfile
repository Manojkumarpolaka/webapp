pipeline {
  agent none
    stages {
        stage ('checkout scm') {
            agent { label 'master' }
            steps {
                checkout(scm)
            }
        }

        stage ('build & push') {
        agent { label 'master' }
            steps {
                sh 'mvn  clean package'
                sh 'ansible-playbook -i hosts dockerimage.yml .' 
            }
        }
    }
}