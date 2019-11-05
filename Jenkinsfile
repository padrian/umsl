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
        //   docker.withRegistry('https://918667847033.dkr.ecr.us-east-2.amazonaws.com/halatech/spps-dash','ecr:us-east-2:ecr-auth') {
        //     def customImage = docker.build("halatech/spps-dash:${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
        //     customImage.push()
        //   }
          echo "here we are"
        }

        stage('Deploy To Staging'){
            checkout scm

            // sh 'wget https://storage.googleapis.com/kubernetes-helm/helm-v2.13.1-linux-amd64.tar.gz && tar -xvf helm-v2.13.1-linux-amd64.tar.gz && mv linux-amd64/helm .'
            // sh './helm lint --strict spps-chart'
            // sh './helm package spps-chart --save=false'
              
            // input(message: "Deploy to Staging ?")

            // withCredentials(bindings: [sshUserPrivateKey(credentialsId: '38285bc6-8bdc-41d6-b980-bd90c85489df', keyFileVariable: 'AWSKEY')]) {
            //     sh 'ssh -oStrictHostKeyChecking=accept-new 18.218.22.90 uptime || true'
            //     sh 'scp -i $AWSKEY spps-chart-0.1.0.tgz admin@18.218.22.90:/tmp/'
            //     sh 'scp -i $AWSKEY spps-chart/configmap-production.yaml admin@18.218.22.90:/tmp/'
            //     sh 'ssh -i $AWSKEY admin@18.218.22.90 kubectl apply -f /tmp/configmap-production.yaml -n production'

            //     sh 'ssh -i $AWSKEY admin@18.218.22.90 helm upgrade --install spps-dash-production /tmp/spps-chart-0.1.0.tgz --set image.tag=$BRANCH_NAME-$BUILD_NUMBER --namespace production --dry-run'
            //     sh 'ssh -i $AWSKEY admin@18.218.22.90 helm upgrade --install spps-dash-production /tmp/spps-chart-0.1.0.tgz --set image.tag=$BRANCH_NAME-$BUILD_NUMBER --namespace production'

            // }
        } // deploy Production   
} //node