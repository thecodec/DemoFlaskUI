def doc_img
pipeline {
    environment {
        //registry = "codctech/flask-ui-demo" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registry = "192.168.60.80:8083/devops-lab-docker-repo"
        //registryCredential = 'codec-docker-hub-login'
        registryCredential = 'devops-lab-nexus'
        dockerImage = ''
    }
    
    agent any
    
    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/thecodec/DemoFlaskUI.git'
            }
        }

        stage ('Stop previous running container'){
            steps{
                sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${JOB_NAME} | awk \'{print $1}\')'
                sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force' //this will delete all images
                sh returnStatus: true, script: 'docker rm ${JOB_NAME}'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    doc_img = registry + ":${env.BUILD_ID}"
                    println ("${doc_img}")
                    dockerImage = docker.build("${doc_img}")
                }
            }
        }

        stage('Test - Run Docker Container on Jenkins node') {
           steps {

                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 5000:5000 ${doc_img}"
            }
        }

        stage('Push To DevOps lab Private Repo') {
            steps {
                script {
                    docker.withRegistry( 'http://' + registry, registryCredential ) {
                    //docker.withRegistry( 'https://registry.hub.docker.com ', registryCredential ) {
                    //withCredentials([string(credentialsId: 'devops_lab_docker_repo_login_pass', variable: 'devops_lab_docker_repo_pass')]) {
                        //sh '''
                        //docker login -u admin -p ${devops_lab_docker_repo_pass} 192.168.60.80:8083
                        dockerImage.push("${env.BUILD_ID}")
                        
                        //dockerImage.rmi("latest")
                        
                        //sh ' sudo docker rmi '
                        //'''
                    }   
                }
            }
        }

        stage('Deploy to DevOps Lab Test Server') {
            steps {
                script {
                    def stopcontainer = "docker stop ${JOB_NAME}"
                    def delcontName = "docker rm ${JOB_NAME}"
                    def delimages = 'docker image prune -a --force'
                    def drun = "docker run -d --name ${JOB_NAME} -p 5000:5000 ${doc_img}"
                    println "${drun}"
                    sshagent(['deploy-server-docker-login']) {
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no vagrant@192.168.60.70 ${stopcontainer} "
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no vagrant@192.168.60.70 ${delcontName}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no vagrant@192.168.60.70 ${delimages}"

                        // some block
                        //sh "ssh -o StrictHostKeyChecking=no vagrant@192.168.60.70 ${drun}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no vagrant@192.168.60.70 ${drun}"
                        
                    }
                }
            }
        }

    }
}