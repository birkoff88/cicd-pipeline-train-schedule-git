// Jenkinsfile — build locally, deploy via SSH (no systemctl)
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        echo 'Running build automation'
        sh '''
          set -euo pipefail
          ./gradlew clean build --no-daemon
          mkdir -p dist
          # Create the zip if your build doesn't already produce it
          if [ ! -f dist/trainSchedule.zip ]; then
            if [ -d build ]; then
              zip -r dist/trainSchedule.zip build/ >/dev/null
            else
              zip -r dist/trainSchedule.zip . -x "dist/*" ".git/*" >/dev/null
            fi
          fi
        '''
        archiveArtifacts artifacts: 'dist/trainSchedule.zip', fingerprint: true, onlyIfSuccessful: true
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
                configName: 'staging', // must match server in Manage Jenkins > Configure System
                sshCredentials: [ username: "${USERNAME}", encryptedPassphrase: "${USERPASS}" ],
                transfers: [
                  // Upload artifact to /tmp on the remote
                  sshTransfer(
                    sourceFiles: 'dist/trainSchedule.zip',
                    removePrefix: 'dist/',
                    remoteDirectory: '/tmp'
                  ),
                  // Unpack to target dir (no systemctl)
                  sshTransfer(
                    execCommand: "sudo mkdir -p /opt/train-schedule && sudo rm -rf /opt/train-schedule/* && unzip -o /tmp/trainSchedule.zip -d /opt/train-schedule && ls -la /opt/train-schedule"
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
