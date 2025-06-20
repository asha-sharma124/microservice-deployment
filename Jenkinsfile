// pipeline{
//     agent any 
//      environment {
//         EC2_IP = credentials('ec2-IP')
//         SSH_KEY = credentials('ssh-key2') // Jenkins credentials me save kiya hua
//         DOCKER_APP_PATH = credentials('dir')
//         USER=credentials('ec2-user2')
//     }
//     stages{
//         stage('build'){
//             steps{
//                 echo "Building the project"
//                 script{
//                     def services =['config-server','discovery-service','card-service','card-statement-composite','edge-server','monitor-dashboard','statement-service','turbine']
//                     for (svc in services){
//                         dir("${svc}"){
//                             echo "building"
//                             sh  'mvn clean package -DskipTests' 
//                         }
//                     }
                
//             }}

//         }
//         stage('COPY to EC2'){
//             steps{
//                 echo "sending file to server"
//                 sh """
//                 chmod 600 ${SSH_KEY}
//                 rsync -avz -e "ssh -i ${SSH_KEY}" ./ ${USER}@${EC2_IP}:${DOCKER_DIR}
//                 """
//             }
//         }
//           stage('Build Docker on EC2') {
//             steps {
//                 echo 'SSH into EC2 and build docker images...'
//                 sh """
//                     ssh -i ${SSH_KEY} ec2-user@${EC2_IP} << EOF
//                         cd ${DOCKER_APP_PATH}
//                         docker-compose build
//                         docker-compose up -d
//                     EOF
//                 """
//             }
//         }


     pipeline {
    agent any

    environment {
        EC2_IP = credentials('ec2-IP')                 // EC2 Public IP (Jenkins credentials)
        SSH_KEY = credentials('ssh-key2')              // EC2 private key stored in Jenkins credentials
        DOCKER_APP_PATH = credentials('dir')           // Directory on EC2 (e.g., /home/ec2-user/spring-microservices)
        USER = credentials('ec2-user')                 // Typically ec2-user or ubuntu
    }

    stages {

        stage('Copy Code to EC2') {
            steps {
                echo "🔄 Sending code to EC2..."
                sh """
                    chmod 600 ${SSH_KEY}
                    rsync -avz -e "ssh -o StrictHostKeyChecking=no -i ${SSH_KEY}" ./ ${USER}@${EC2_IP}:${DOCKER_APP_PATH}
                """
            }
        }

        stage('Build Project with Maven on EC2') {
            steps {
                echo "⚙️ Building project on EC2..."
                sh """
                    ssh -T -o StrictHostKeyChecking=no -i ${SSH_KEY} ${USER}@${EC2_IP} << 'EOF'
cd ${DOCKER_APP_PATH}
for svc in config-server discovery-service card-service card-statement-composite edge-server monitor-dashboard statement-service turbine; do
    echo "🚧 Building \$svc"
    cd \$svc
    mvn clean package -DskipTests
    cd ..
done
EOF
                """
            }
        }

        stage('Build & Run Docker on EC2') {
            steps {
                echo "🐳 Building Docker images and running containers..."
                sh """
                    ssh -T -o StrictHostKeyChecking=no -i ${SSH_KEY} ${USER}@${EC2_IP} << 'EOF'
cd ${DOCKER_APP_PATH}
docker compose build
docker compose up -d
EOF
                """
            }
        }
    }
}
