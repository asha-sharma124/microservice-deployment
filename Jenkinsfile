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


    //  pipeline {
    // agent any

    // environment {
    //     EC2_IP = credentials('ec2-IP')                 // EC2 Public IP (Jenkins credentials)
    //     SSH_KEY = credentials('ssh-key2')              // EC2 private key stored in Jenkins credentials
    //     DOCKER_APP_PATH = credentials('dir')           // Directory on EC2 (e.g., /home/ec2-user/spring-microservices)
    //     USER = credentials('ec2-user')                 // Typically ec2-user or ubuntu
    // }

    // stages {
    //    stage('Checkout Code') {
    //         steps {
    //             checkout scm
    //         }
    //     }

        // stage('Copy Code to EC2') {
        //     steps {
        //         echo "🔄 Sending code to EC2..."
        //         sh """
        //             chmod 600 ${SSH_KEY}
        //             rsync -avz -e "ssh -o StrictHostKeyChecking=no -i ${SSH_KEY}" ./ ${USER}@${EC2_IP}:${DOCKER_APP_PATH}
        //         """
        //     }
        // }
pipeline {
  agent any

  environment {
    DOCKERHUB_USERNAME = credentials('docker-user')
    DOCKERHUB_PASSWORD = credentials('docker-pass')
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Build All Services with Maven') {
      steps {
        echo "⚙️ Building all microservices..."
        script {
          docker.image('maven:3.9.6-eclipse-temurin-17').inside {
            sh '''
              mvn -v  # 👈 Verify Maven inside the container

              for svc in config-server discovery-service card-service card-statement-composite edge-server monitor-dashboard statement-service turbine; do
                echo "🚧 Building $svc"
                cd $svc
                mvn clean package -DskipTests
                cd ..
              done
            '''
          }
        }
      }
    }

    stage('Login to DockerHub') {
      steps {
        sh '''
          echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
        '''
      }
    }

    stage('Generate Image Tag') {
      steps {
        script {
          def tag = new Date().format("yyyyMMdd-HHmmss")
          env.IMAGE_TAG = tag
        }
      }
    }

    stage('Build & Push Docker Images') {
      steps {
        sh '''
          for svc in config-server discovery-service card-service card-statement-composite edge-server monitor-dashboard statement-service turbine; do
            echo "🐳 Building image for $svc"
            docker build -t $DOCKERHUB_USERNAME/$svc:$IMAGE_TAG ./$svc
            docker push $DOCKERHUB_USERNAME/$svc:$IMAGE_TAG
          done
        '''
      }
    }

    stage('Update docker-compose.yml') {
      steps {
        sh '''
          for svc in config-server discovery-service card-service card-statement-composite edge-server monitor-dashboard statement-service turbine; do
            echo "🔧 Updating image tag for $svc in docker-compose.yml"
            sed -i "s|image: $DOCKERHUB_USERNAME/$svc:.*|image: $DOCKERHUB_USERNAME/$svc:$IMAGE_TAG|" docker-compose.yml
          done
        '''
      }
    }

    stage('Pull & Start All Services') {
      steps {
        sh '''
          docker-compose pull
          docker-compose up -d
        '''
      }
    }
  }

  post {
    success {
      echo "✅ All services deployed with tag: $IMAGE_TAG"
    }
    failure {
      echo "❌ Deployment failed."
    }
  }
}
