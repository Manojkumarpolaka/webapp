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
                sh '/usr/local/apache-maven-3.8.5/bin/mvn clean package'
                sh 'ansible-playbook -i hosts dockerimage.yml' 
            }
        }
    }
}