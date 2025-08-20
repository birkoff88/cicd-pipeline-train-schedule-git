pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        echo 'Running build automation'
      }
    }

    stage('DeployToStaging') {
      when { branch 'master' } // change to 'main' if needed
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
                configName: 'staging', // must match the server name in Jenkins global config
                sshCredentials: [
                  username: "${USERNAME}",
                  encryptedPassphrase: "${USERPASS}" // used as password when no key/keyPath is set
                ],
                transfers: [
                  sshTransfer(
                    // Just run a command remotely to prove auth works
                    execCommand: "touch /tmp/jenkins_auth_test_${BUILD_NUMBER} && echo \"created by Jenkins at $(date)\" >> /tmp/jenkins_auth_test_${BUILD_NUMBER}",
                    usePty: true // set to false if your sudoers/TTY isn't required
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

