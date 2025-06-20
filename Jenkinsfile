pipeline{
    agent any 
     environment {
        EC2_IP = credentials('ec2-IP')
        SSH_KEY = credentials('ssh-key2') // Jenkins credentials me save kiya hua
        DOCKER_APP_PATH = credentials('dir')
        USER=credentials('ec2-user2')
    }
    stages{
        stage('build'){
            steps{
                echo "Building the project"
                script{
                    def services =['config-server','discovery-service','card-service','card-statement-composite','edge-server','monitor-dashboard','statement-service','turbine']
                    for (svc in services){
                        dir("${svc}"){
                            echo "building"
                            sh  'mvn clean package -DskipTests' 
                        }
                    }
                
            }}

        }
        stage('COPY to EC2'){
            steps{
                echo "sending file to server"
                sh """
                chmod 600 ${SSH_KEY}
                rsync -avz -e "ssh -i ${SSH_KEY}" ./ ${USER}@${EC2_IP}:${DOCKER_DIR}
                """
            }
        }
          stage('Build Docker on EC2') {
            steps {
                echo 'SSH into EC2 and build docker images...'
                sh """
                    ssh -i ${SSH_KEY} ec2-user@${EC2_IP} << EOF
                        cd ${DOCKER_APP_PATH}
                        docker-compose build
                        docker-compose up -d
                    EOF
                """
            }
        }


    }
}
