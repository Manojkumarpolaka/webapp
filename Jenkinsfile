pipeline {
  agent none
    stages {
        stage ('checkout scm') {
            agent { label 'master' }
            steps {
                git url: 'https://github.com/Manojkumarpolaka/webapp.git', branch: "master"
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