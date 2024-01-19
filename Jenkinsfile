    pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "ujebpathan/train-schedule"
        docker_hub_login = 'docker-hub-credentials-id'
        JAVA_HOME = tool 'java-8'
        KUBECONFIG_CREDS = credentials('kubeconfig-credentials-id')
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            //when {
            //    branch 'master'
           // }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
          //  when {
          //      branch 'master'
          //  }
            steps {
                script {
                    //docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                      //  app.push("${env.BUILD_NUMBER}")
                      //  app.push("latest")
                    docker.withRegistry('',docker_hub_login) {
                        def app = docker.image(DOCKER_IMAGE_NAME)
                        app.push("latest")
                    }
                }
            }
        }
       // stage('CanaryDeploy') {
          //  when {
           //     branch 'master'
           // }
        //    environment { 
          //      CANARY_REPLICAS = 1
          //  }
          //  steps {
           //     kubernetesDeploy(
            //        kubeconfigId: 'kubeconfig',
             //       configs: 'train-schedule-kube-canary.yml',
              //      enableConfigSubstitution: true
              //  )
           // }
      //  }
        stage('DeployToProduction') {
             steps{
                script {
                //  sh "sed -i 's,TEST_IMAGE_NAME,harshmanvar/node-web-app:$BUILD_NUMBER,' deployment.yaml"
                    sh "cat train-schedule-kube.yml"
                    sh """
                        echo '${KUBECONFIG_CREDS}' > /tmp/kubeconfig
                    """

                    // Set KUBECONFIG environment variable
                    withCredentials([string(credentialsId: 'kubeconfig-credentials-id', variable: 'KUBECONFIG_CREDS')]) {

                        // Your kubectl commands here
                        sh "kubectl get pods"
                         sh "kubectl --kubeconfig=/home/ec2-user/config apply -f train-schedule-kube.yml"

                    }
                }    
            }
        }
    }
}


