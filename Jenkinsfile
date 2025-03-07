pipeline { 

    agent any 

 

    environment { 

        CONTAINER_NAME_F = 'educationfront'  
        CONTAINER_NAME_B = 'educationback'


        IMAGE_NAME = 'image'          // Docker image name 

        DOCKER_REGISTRY = 'docker.io'  // Docker registry (Docker Hub) 

        DOCKER_REPO = 'jagadishspyd/e-education' // Replace with your Docker Hub username/repository 

        TAG_F = "1.0"  // Tag for the Docker image (e.g., latest or versioned)
        TAG_B = "1.0BE"

    } 

 

    stages { 

        stage ('Installing dependices'){ 

             steps { 

                echo 'Installing.....' 

                sh 'sudo yum install git -y' 

                sh 'sudo yum install docker -y' 

                sh 'sudo systemctl enable docker' 

                sh 'sudo systemctl start docker' 

                sh 'sudo usermod -aG docker jenkins' 

                //sh 'sudo systemctl restart jenkins' 

            } 

        } 

        stage('Clone From Git') { 

            steps { 

                echo 'Cloning repository from GitHub' 

                git branch: 'main', url: 'https://github.com/spydtech/E-education-frontend.git' 

            } 

        } 

 

        stage('Build a Docker Image') { 

            steps { 

                echo 'Building Docker image' 

                sh 'docker build -t $DOCKER_REPO:$TAG_F .' 

            } 

        } 

 

        //stage('Build a Docker Container') { 

      // steps { 

               // echo 'Running Docker container' 

                //sh 'docker stop $CONTAINER_NAME ' 

                //sh 'docker rm $CONTAINER_NAME' 

                //sh 'docker run -d --name $CONTAINER_NAME -p 80:80 $DOCKER_REPO:$TAG' 

          //  } 

       // } 

 

        stage('Push to Docker Hub') { 

            steps { 

                script { 

                    // Login to Docker Hub using the credentials 

                    withCredentials([usernamePassword(credentialsId: 'eeducation', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) { 

                        // Login to Docker Hub 

                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin' 

                    } 

 

                    // Push the Docker image to Docker Hub 

                    echo 'Pushing Docker image to Docker Hub' 

                    sh 'docker push $DOCKER_REPO:$TAG_F' 

                } 

            } 

        } 

 

        stage('Deleting container and Images') { 

            steps { 

                echo 'Deleting container and images...' 

                sh 'docker rm -f $CONTAINER_NAME_F || true' 

                sh 'docker rm -f $CONTAINER_NAME_F || true' 

                sh 'docker rmi $DOCKER_REPO:$TAG_F || true' 

               // sh 'docker system prune -af || true' 

            } 

        } 
        
        stage('Cleanup Docker Space') {
            steps {
                script {
                    sh '''
                    echo "Stopping Docker services..."
                    sudo systemctl stop docker.socket || true
                    sudo systemctl stop docker || true

                    echo "Cleaning up Docker data..."
                    sudo rm -rf /var/lib/docker || true

                    echo "Restarting Docker service..."
                    sudo systemctl start docker

                    echo "Verifying Docker status..."
                    sudo systemctl status docker || true
                    '''
                }
            }
        }

 

        stage('Pull the Docker image') { 

            steps { 

                echo 'Pulling Docker image from Docker Hub' 

                sh 'docker pull $DOCKER_REPO:$TAG_F'
                sh 'docker pull $DOCKER_REPO:$TAG_B'

            } 

        } 

 

        stage('Create a New Container') { 

            steps { 

                echo 'Creating a new Docker container' 

                sh 'docker run -d --name $CONTAINER_NAME_F -p 80:80 $DOCKER_REPO:$TAG_F'
                sh 'docker run -d --name $CONTAINER_NAME_B -p 8082:81 $DOCKER_REPO:$TAG_B'

            } 

        } 
        
        stage('Cleanup Dependencies') { 
            steps { 
                echo 'Cleaning up all dependencies except Docker images and containers...'
                sh '''
                sudo yum remove -y git
                sudo yum clean all
                sudo rm -rf /var/cache/yum
                sudo rm -rf /tmp/*
                '''
            } 
        }
        
        stage('Creating Artifacts') { 

            steps { 

                echo 'Creating a Artifactsr' 

                archiveArtifacts artifacts: '*/**', followSymlinks: false, onlyIfSuccessful: true 

            } 

        } 

    } 

 

    post { 

        success { 

            echo "Pipeline succeeded!" 

        } 

 

        failure { 

            echo "Pipeline failed!" 

        } 

    } 

} 
