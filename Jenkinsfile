node {
    //URL to Github repository https://github.com/<owner>/<repo>
    def GITHUB_REPO_URL="https://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@github.com/rslangehennig/aceivt-frontend"
    //Retrieve ENV ID values for DEV and QA from "Download attributes from API" json file
    //def VELOCITY_ENV_ID_DEV="a4a91beb-e2be-4da5-896a-78632d3b451c"
    //VELOCITY_APP_NAME must match your Velocity pipeline application name
    //def VELOCITY_APP_NAME="Online-Botanicals"
    //Version number
    def VERSION_NUMBER="2.1"
    //Do not change below this line.
    def GIT_COMMIT

    stage ('cloning the repository'){
        currentBuild.displayName = "${VERSION_NUMBER}.${BUILD_NUMBER}"
        echo currentBuild.displayName = "${currentBuild.displayName}"
        majorVersion="${BUILD_NUMBER}"
        def scm = git branch: 'main', url: "${GITHUB_REPO_URL}"
        GIT_COMMIT = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
        echo "GIT_COMMIT=${GIT_COMMIT}"
    }

    stage ('build') {
        sh '''#!/bin/bash
          echo $WORKSPACE
          pwd
          ls -al
          
          echo "Checking for changes"
          git diff --name-only $GIT_PREVIOUS_COMMIT $GIT_COMMIT
          git diff --name-only $GIT_PREVIOUS_COMMIT $GIT_COMMIT > diff.output
          ls -al
          cat diff.output
          wc -l diff.output
          lineCount=`wc -l diff.output | cut -f1 -d' '`
          echo lineCount is $lineCount

          if [ $lineCount = "0" ]
          then
              echo "No file changes found"
          else
              echo "File changes identified..."
              # Create new component version
              echo "Create new component version"
              rm -f diff.output
              /opt/IBM/udclient/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN createVersion -component CP4I-ACEIVT-API -name $CURRENT_BUILD
              /opt/IBM/udclient/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN addVersionLink -component CP4I-ACEIVT-API -version $CURRENT_BUILD -linkName "Jenkins Pipeline" -link http://52.116.6.138:8081/job/ace-ivt-frontend/${BUILD_NUMBER}/
              /opt/IBM/udclient/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN addVersionFiles -component CP4I-ACEIVT-API -version $CURRENT_BUILD -base "/opt/jenkins-agent/workspace/ace-ivt-pipelines/ace-ivt-frontend-pipeline"

              # Add a component status to allow it to pass gates in UCD
              #echo "Adding component status"
              #/opt/IBM/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN addVersionStatus -component Online-Botanical-APP -version $CURRENT_BUILD -status "Unit Tests Passed"

              # Create snapshot
              #echo "Create new snapshot"
              #cp urbancode/snapshot.json .
              #sed -i "s/MYNAME/$CURRENT_BUILD/g" snapshot.json
              #cat snapshot.json
              #/opt/IBM/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN createSnapshot snapshot.json
          fi
          
        '''
    }

    stage ('Push Artifact of Build to UCD and Trigger Deploy') {
         sh '''#!/bin/bash

            # Run udclient to trigger deployment
            pwd
            ls -al

            rm -f README.md
            rm -rf .git
            rm -rf jenkins
            cp urbancode/deploy.json ..
            rm -rf urbancode

            pwd
            ls -al

            export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/
            export UCD_AUTH_TOKEN="9ec486c6-c7bc-4040-ac1d-f6b6b3e040c7"
            export UCD_WEB_URL="https://169.62.212.57:8444"

            # Deploy snapshot to DEV environment
            #cp urbancode/deploy-snapshot.json .
            #sed -i "s/MYNAME/$CURRENT_BUILD/g" deploy-snapshot.json
            #cat deploy-snapshot.json

            /opt/IBM/udclient/udclient -weburl $UCD_WEB_URL -authtoken $UCD_AUTH_TOKEN requestApplicationProcess ../deploy.json

            rm -f ../deploy.json
            echo All done
            '''
    }
}
