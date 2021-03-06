pipeline {
    agent none
    environment {
        imageName = 'navigative/demo'
        port = 80
    }

    stages {
       stage('Build') {
          agent {label 'mgr1'}
          steps {
              sh "docker --version"
              sh "sudo usermod -aG docker jenkins"
              sh "docker build -t ${env.imageName} ."
          }
       }

       stage('Package') {
          agent {label 'mgr1'}
          steps {
            withCredentials(
                [usernamePassword(
                    credentialsId: 'docker_hub',
                    passwordVariable: 'dockerHubPassword',
                    usernameVariable: 'dockerHubUser'
                )]
            ){
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                sh "docker push ${env.imageName}"
            }
          }
       }

       stage('Deploy') {
          agent {label 'mgr1'}
          steps {
              script {
                  try {
                    sh "docker service update --image ${env.imageName} demo"
                    sh "echo update service"
                  } catch (e){
                    sh "docker service create --name demo -p ${env.port}:80 ${env.imageName}"
                    sh "echo create service"
                  }
              }
          }
       }
    }
}
