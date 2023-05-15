pipeline {
    agent {
      kubernetes {
        yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: node
            image: node:16
            command:
            - cat
            tty: true
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
          '''
        }
    }
    environment {
        DEV_NAME = "backend-dev-rnpl"
        PROD_NAME = "rnpl-prod"
        VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
        IMAGE_REPO = "spleet"
    }
    stages {
       stage("Checkout code"){
            steps{
                container('node'){
                 checkout scm
                }
            }
        }
        stage('Build and Test'){
            // when{
            //     branch 'dev'
            // }
            steps{
                container('node'){
                    sh "npm install"
                    sh "npm run test"
                }
            }
        }
        stage('Build Dev Image') {
            when{
                branch 'dev'
            }
            steps {
                container('docker') {
                   sh "docker build -t ${DEV_NAME} ."
                   sh "docker tag ${DEV_NAME}:latest ${IMAGE_REPO}/${DEV_NAME}:${VERSION}"
                }
            }
        }
        stage('Push Dev Image'){
            when{
              branch 'dev'
            }
            steps{
                container('docker'){
                    withDockerRegistry(credentialsId: 'dockerhub', url: ''){
                      sh "docker push ${IMAGE_REPO}/${DEV_NAME}:${VERSION}"
                    }
                }
                
            }
        }
        stage('Build Prod Image'){
              when{
               branch 'main'
            }
            steps{
                container('docker'){
                    sh "docker build -t ${PROD_NAME} ."
                    sh "docker tag ${PROD_NAME}:latest ${IMAGE_REPO}/${PROD_NAME}:${VERSION}"

                }
            }
        }
        stage('Push Prod Image') {
            when{
             branch 'main'
            }
            steps {
               container('docker') {
                  withDockerRegistry(credentialsId: 'dockerhub', url: ''){
                       sh "docker push ${IMAGE_REPO}/${PROD_NAME}:${VERSION}"
                    }
                }
            }
        }

        stage("Trigger Upate For Dev"){
              when{
                  branch 'dev'
                }
            steps{
                container('docker') {
                   echo "Triggering Manifest Job"
                
                   build job: 'rnpl-dev-manifest', parameters: [string(name: 'VERSION', value: env.VERSION)]

                }

        }   }
         
        stage("Trigger Update for Prod"){
               when{
                   branch 'main'
                }
            steps{
                   container('docker'){
                   build job: 'rnpl-prod-manifest', parameters: [string(name: 'VERSION', value: env.VERSION)]
                }  
            }
        }
        stage("Notify Slack"){
            steps{
                notifyUsers()
            }
        }
        // post{
        //     // success{
        //     //     slackSend channel: 'spleetafrica', message: 'Build Succeeded', tokenCredentialId: 'slack-notification'
        //     // }
        //     failure{
        //         slackSend failOnError:true message:{Build failed  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'slack-notification'
        //     }
        // }

       
    }
     
}
