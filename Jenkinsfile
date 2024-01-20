 pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "ujebpathan/train-schedule"
        docker_hub_login = 'docker-hub-credentials-id'
        JAVA_HOME = tool 'java-8'
        KUBECONFIG_CREDS = credentials('k8s-id')
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
                    docker.withRegistry('https://registry.hub.docker.com',docker_hub_login) {
                        def app = docker.image(DOCKER_IMAGE_NAME)
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
       //     when {
        //        branch 'master'
        //    }
            environment { 
              CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'k8s-id',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
           // when {
           //     branch 'master'
           // }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'k8s-id',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'k8s-id',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
