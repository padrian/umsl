#!/usr/bin/groovy
node {

        stage('Checkout') {
          echo "Checkout Source Code"
          checkout scm       
        }
        
        stage('Tests') {
          sh './mvnw clean -X verify'
        }

        stage('Docker Build') {
          echo "Build Container and Push to AWS ECR"
          docker.withRegistry('https://366670401047.dkr.ecr.us-east-2.amazonaws.com/future-airlines/umsl','ecr:us-east-2:aws-ecr-auth') {
            def customImage = docker.build("future-airlines/umsl:${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
            customImage.push()
          }   
        }

        stage('Deploy To Staging'){

          sh 'wget https://storage.googleapis.com/kubernetes-helm/helm-v2.13.1-linux-amd64.tar.gz && tar -xvf helm-v2.13.1-linux-amd64.tar.gz && mv linux-amd64/helm .'
          sh './helm lint --strict umsl-chart'
          sh './helm package umsl-chart --save=false'
              
          withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'aws-ssh-access', keyFileVariable: 'AWSKEY')]) {
          sh 'ssh -oStrictHostKeyChecking=accept-new 3.133.93.101 uptime || true'
          sh 'scp -i $AWSKEY umsl-chart-0.1.0.tgz ubuntu@3.133.93.101:/tmp/'
          sh 'scp -i $AWSKEY umsl-chart/configmap-production.yaml ubuntu@3.133.93.101:/tmp/'
          sh 'ssh -i $AWSKEY ubuntu@3.133.93.101 kubectl apply -f /tmp/configmap-production.yaml -n production'

          sh 'ssh -i $AWSKEY ubuntu@3.133.93.101 helm upgrade --install umsl-dash-production /tmp/umsl-chart-0.1.0.tgz --set image.tag=$BRANCH_NAME-$BUILD_NUMBER --namespace production --dry-run'
          sh 'ssh -i $AWSKEY ubuntu@3.133.93.101 helm upgrade --install umsl-dash-production /tmp/umsl-chart-0.1.0.tgz --set image.tag=$BRANCH_NAME-$BUILD_NUMBER --namespace production'

         }
        } // deploy Production   
} //node