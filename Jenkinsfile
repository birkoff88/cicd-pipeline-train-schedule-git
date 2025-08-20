// Jenkinsfile
pipeline {
  agent any

  stages {
    stage('Auth & Touch') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'webserver_login',
          usernameVariable: 'USERNAME',
          passwordVariable: 'USERPASS'
        )]) {
          sshPublisher(
            failOnError: true,
            continueOnError: false,
            publishers: [
              sshPublisherDesc(
                configName: 'staging', // this server must exist in Manage Jenkins > Configure System
                // If your server is already configured with creds, you can drop sshCredentials below.
                sshCredentials: [ username: "${USERNAME}", encryptedPassphrase: "${USERPASS}" ],
                transfers: [
                  sshTransfer(
                    execCommand: "touch /tmp/jenkins_auth_ok_${env.BUILD_NUMBER}"
                  )
                ],
                verbose: true
              )
            ]
          )
        }
      }
    }
  }
}
