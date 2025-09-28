pipeline {
    agent any

    tools {
        maven 'maven 3.9.11' // Ensure Maven is installed and configured in Jenkins        
    }

    environment {
        DOCKER_IMAGE = 'demo-app'
        CONTAINER_NAME = 'springboot-container'
        JAR_FILE_NAME = 'app.jar'
        PORT = '8081'
        REMOTE_USER = "ec2-user"
        REMOTE_HOST = "54.180.147.37"
        REMOTE_DIR = "/home/ec2-user/deploy"
        SSH_CREDENTIALS_ID = "4bb8df04-f6ed-4eaf-81a7-2521b696abfd" // Replace with your Jenkins SSH credentials ID
    }    

    stages {
        
        stage('Git Checkout ') {
            steps {
                echo 'Checking out source code...'
                checkout scm //Jenkines가 연결된 git 저장소에서 최신코드 체크아웃
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Building...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Prepare Jar') {
            steps {
                echo 'Prepare Jar...'
                sh 'cp target/demo.0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        } 

        stage('Copy to Remote Server') {
            steps {
                echo 'Remoting...'
                sshagent (credentials: [environment.SSH_CREDENTIALS_ID]) {
                    sh "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} \"mkdir -p ${REMOTE_DIR}\""
                    // JAR 파일과 Dockerfile을 원격 서버에 복사
                    sh "scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${JAR_FILE_NAME} Dockerfile ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
                }                
            }
        }

        stage('Remote Docker Build & Deploy') {
            steps {
                echo 'Remote Docker Build & Deploy...'
                sshagent (credentials: [environment.SSH_CREDENTIALS_ID]) {                   
                     // 원격 서버에서 도커 컨테이너를 제거하고 새로 빌드 및 실행
                    sh """
                    ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} << ENDSSH
                        cd ${REMOTE_DIR} || exit 1                          # 복사한 디렉토리로 이동
                        docker rm -f ${CONTAINER_NAME} || true             # 이전에 실행 중인 컨테이너 삭제 (없으면 무시)
                        docker build -t ${DOCKER_IMAGE} .                  # 현재 디렉토리에서 Docker 이미지 빌드
                        docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE} # 새 컨테이너 실행
                    ENDSSH
                    """
                }                
            }
        }
    }
}