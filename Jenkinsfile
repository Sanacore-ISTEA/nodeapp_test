pipeline {
  agent any
    environment{
      ServidorMaster="ServidorSSH"
      ServidorDeploy="192.168.0.79"
      PathDeploy="/home/to_implement"
  }
    stages {
          stage('Install dependencies') {
        steps {
          sh 'npm install'
        }
      }
      stage('test'){
        steps {
          sh 'npm run build'
        }
      }
    stage('Delivery to artifact in server'){
      steps{
        //sshPublisher(publishers: [sshPublisherDesc(configName: 'ServidorSSH', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home/to_implement', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        sshPublisher(publishers: [sshPublisherDesc(configName: "${env.ServidorMaster}", transfers: [sshTransfer(cleanRemote: true, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/war', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        sshPublisher(publishers: [sshPublisherDesc(configName: "${env.ServidorMaster}", transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/config', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.config')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
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
            //DEPLOY TO WAR
            writeFile file: 'deploy.sh', text: "cp -i /home/to_implement/war/*.war /opt/tomcat/apache-tomcat-9.0.64/webapps \n sleep 2m"
            sshScript remote: remote, script: 'deploy.sh'
            sshCommand remote: remote, command: "/opt/tomcat/apache-tomcat-9.0.64/bin/shutdown.sh"
            sshCommand remote: remote, command: "/opt/tomcat/apache-tomcat-9.0.64/bin/startup.sh"

          }
        }
      }
    }
  }
}
