pipeline {
  agent any
    environment{
      ServidorMaster="ServidorSSH"
      ServidorDeploy="192.168.0.79"
      PathDeploy="/home/to_implement"
      DOCKERHUB_CREDENTIALS = credentials ('userDocker')
      RepoDockerHub = 'sanacoreistea'
      NameContainerApp= 'miPagina'
      NameImagenDockerApp = 'imageMipagina'
  }
    stages {
      stage('Build'){
        steps{
          sh "docker build -t ${env.RepoDockerHub}/${env.NameImagenDockerApp}:${env.BUILD_NUMBER} ."
        }
      }
  
      stage('Login'){
        steps{
          sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password_stdin"
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
            remote.host = "${env.ServidorDeploy}"
            remote.allowAnyHosts = true
            withCredentials([sshUserPrivateKey(credentialsId: 'sshUser-key', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
              remote.user = userName
              remote.identityFile = identity
              //LOGIN TO REGISTRY
              writeFile file: "login.sh", text: "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password_stdin"
              sshScript remote: remote, script: "login.sh"

              // PARAR EL CONTENEDOR
              writeFile file: "login.sh", text: "#!/bin/sh \n docker stop ${env.NameContainerDocker} \n exit 0"
              sshScript remote: remote, script: "stop.sh"              
              
              // BORRAR EL CONTENEDOR
              writeFile file: "rm.sh", text: "#!/bin/sh \n docker rm ${env.NameContainerDocker} \n exit 0"
              sshScript remote: remote, script: "rm.sh"

              // DEPLOTAR LA IMAGEN DOCKER
              writeFile file: 'deploy.sh', text: "docker run -d --name ${env.NameContainerDocker} -p 3000:3000 ${env.RootDockerHub}/nodeapp"
              sshScript remote: remote, script: "deploy.sh"

              // LOGOUT DE DOCKER
              sshCommand remote: remote, command: "docker logout"
              
              //writeFile file: 'deploy.sh', text: "cp -i /home/to_implement/war/*.war /opt/tomcat/apache-tomcat-9.0.64/webapps \n sleep 2m"
              //sshScript remote: remote, script: 'deploy.sh'
              //sshCommand remote: remote, command: "/opt/tomcat/apache-tomcat-9.0.64/bin/shutdown.sh"
              //sshCommand remote: remote, command: "/opt/tomcat/apache-tomcat-9.0.64/bin/startup.sh"

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
