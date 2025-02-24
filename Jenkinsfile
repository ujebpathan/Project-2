 pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "ujebpathan/train-schedule"
        docker_hub_login = 'docker-hub-credentials-id'
        JAVA_HOME = tool 'java-8'
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
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World! 1'
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
      stage("SSH Into k8s Server") {
            steps {
                script {
                    def remote = [:]
                    remote.name = 'K8S master'
                    remote.host = '172.26.11.88'
                    remote.user = 'ujeb'
                    remote.password = 'ujeb'
                    remote.allowAnyHosts = true

                    // Copy the Kubernetes YAML file to the remote server
                    sshPut remote: remote, from: 'train-schedule-kube.yml', into: '.'

                    // Deploy the Kubernetes YAML file using kubectl
                    sshCommand remote: remote, command: "kubectl apply -f train-schedule-kube.yml"
                  }
            }
      }
    }
 }
