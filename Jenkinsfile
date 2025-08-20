// Jenkinsfile
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        echo 'Running build automation'
        sh '''
          set -eu
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
      when { branch 'master' }
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
                configName: 'staging',
                sshCredentials: [ username: "${USERNAME}", encryptedPassphrase: "${USERPASS}" ],
                transfers: [
                  sshTransfer(
                    sourceFiles: 'dist/trainSchedule.zip',
                    removePrefix: 'dist/',
                    remoteDirectory: '/tmp'
                  ),
                  sshTransfer(
                    // no systemctl; just unpack
                    execCommand: "mkdir -p \"$HOME/train-schedule\" && rm -rf \"$HOME/train-schedule\"/* && unzip -o /tmp/trainSchedule.zip -d \"$HOME/train-schedule\" && ls -la \"$HOME/train-schedule\""
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
