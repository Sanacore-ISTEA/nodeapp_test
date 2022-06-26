pipeline {
  agent any
    environment{
      DOCKERHUB_CREDENCIALS = credentials ('userDocker')
      RepoDockerHub = 'sanacoreistea'
      NameContainerApp= 'mi_pagina'
      NameImagenDockerApp = 'image_mi_pagina'
  }
    stages {
      stage('Build'){
        steps{
          sh "docker build -t ${env.RepoDockerHub}/${env.NameImagenDockerApp}:${env.BUILD_NUMBER} ."
        }
      }
  
      stage('Login'){
        steps{
          sh "echo C4t4l1n4! | docker login -u sanacoreistea --password-stdin"
        }
      }

      stage('Push'){
        steps{
          sh "docker push ${env.RepoDockerHub}/${env.NameImagenDockerApp}:${env.BUILD_NUMBER}"
        }
      }
      stage('Deploy to server'){
        steps{
          script{
            def remote = [:]
            remote.name = 'dev'
            remote.host = '192.168.0.79'
            remote.allowAnyHosts = true
            withCredentials([sshUserPrivateKey(credentialsId: 'sshUser-key', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
              remote.user = userName
              remote.identityFile = identity
              
              //LOGIN TO REGISTRY
              writeFile file: "login.sh", text: "echo C4t4l1n4! | docker login -u sanacoreistea --password-stdin"
              sshScript remote: remote, script: "login.sh"

              // PARAR EL CONTENEDOR
              writeFile file: "login.sh", text: "#!/bin/sh \n docker stop ${env.NameContainerApp} \n exit 0"
              sshScript remote: remote, script: "stop.sh"              
              
              // BORRAR EL CONTENEDOR
              writeFile file: "rm.sh", text: "#!/bin/sh \n docker rm ${env.NameContainerApp} \n exit 0"
              sshScript remote: remote, script: "rm.sh"

              // DEPLOTAR LA IMAGEN DOCKER
              writeFile file: 'deploy.sh', text: "docker run -d --name ${env.NameContainerApp} -p 4000:3000 ${env.RepoDockerHub}/${env.NameImagenDockerApp}:${env.BUILD_NUMBER}"
              sshScript remote: remote, script: "deploy.sh"

              // LOGOUT DE DOCKER
              sshCommand remote: remote, command: "docker logout"
              
          }
        }
      }
    post{
      always{
        sh 'docker logout'
      }
    }
    }
  }
}
