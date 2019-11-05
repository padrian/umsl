#!/usr/bin/groovy
node {
    try{
        notifyBuild('STARTED')

        if(env.BRANCH_NAME == 'deploy'){

        stage('Checkout') {
          echo "Let the Builds Begin"
          checkout scm
        }

        stage('Tests') {
          docker.image('openjdk:8').inside {
            withEnv(['HOME=.']){
              ./mvnw clean verify
            }
          }
        }

        stage('Docker Build') {
          docker.withRegistry('https://918667847033.dkr.ecr.us-east-2.amazonaws.com/halatech/spps-dash','ecr:us-east-2:ecr-auth') {
            def customImage = docker.build("halatech/spps-dash:${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
            customImage.push()
          }
        }

        stage('Deploy To Staging'){
            checkout scm

            sh 'wget https://storage.googleapis.com/kubernetes-helm/helm-v2.13.1-linux-amd64.tar.gz && tar -xvf helm-v2.13.1-linux-amd64.tar.gz && mv linux-amd64/helm .'
            sh './helm lint --strict spps-chart'
            sh './helm package spps-chart --save=false'
              
            input(message: "Deploy to Staging ?")

            withCredentials(bindings: [sshUserPrivateKey(credentialsId: '38285bc6-8bdc-41d6-b980-bd90c85489df', keyFileVariable: 'AWSKEY')]) {
                sh 'ssh -oStrictHostKeyChecking=accept-new 18.218.22.90 uptime || true'
                sh 'scp -i $AWSKEY spps-chart-0.1.0.tgz admin@18.218.22.90:/tmp/'
                sh 'scp -i $AWSKEY spps-chart/configmap-production.yaml admin@18.218.22.90:/tmp/'
                sh 'ssh -i $AWSKEY admin@18.218.22.90 kubectl apply -f /tmp/configmap-production.yaml -n production'

                sh 'ssh -i $AWSKEY admin@18.218.22.90 helm upgrade --install spps-dash-production /tmp/spps-chart-0.1.0.tgz --set image.tag=$BRANCH_NAME-$BUILD_NUMBER --namespace production --dry-run'
                sh 'ssh -i $AWSKEY admin@18.218.22.90 helm upgrade --install spps-dash-production /tmp/spps-chart-0.1.0.tgz --set image.tag=$BRANCH_NAME-$BUILD_NUMBER --namespace production'

            }
        } // deploy Production

        } //run only on develop branch

    } catch(e){
        // If there was an exception thrown, the build failed
        currentBuild.result = "FAILED"
        throw e
    } finally {
        notifyBuild(currentBuild.result)
    }

} //node

// slack and email notifications

def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"
  def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
    <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)

  emailext(
      subject: subject,
      body: details,
      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
    )
} //notifyBuild