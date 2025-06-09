pipeline {
    agent any // Sử dụng bất kỳ agent nào (Jenkins agent đang chạy trên EC2 instance này)

    environment {
        // Biến môi trường cho địa chỉ IP công cộng của EC2 (thay bằng IP thực tế của bạn)
        EC2_PUBLIC_IP = '34.239.110.232' // Đảm bảo đây là IP hiện tại của EC2 của bạn
        DOCKER_HUB_USERNAME = 'your_docker_hub_username' // Thay bằng username Docker Hub của bạn
        APP_NAME = 'my-static-website-app' // Tên cho ứng dụng Docker container
        IMAGE_NAME = "${DOCKER_HUB_USERNAME}/${APP_NAME}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                // Bước này tự động được Jenkins thực hiện khi "Pipeline script from SCM" được cấu hình
                script {
                    echo 'Checking out code from Git...'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    // Xây dựng ảnh Docker từ Dockerfile trong thư mục gốc của repo
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    echo 'Logging in to Docker Hub...'
                    // Sử dụng credentials đã cấu hình trong Jenkins
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USER')]) {
                        sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USER} --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo 'Pushing Docker image to Docker Hub...'
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    echo "Deploying ${APP_NAME} to EC2 instance at ${EC2_PUBLIC_IP}..."
                    // Dừng và xóa container cũ nếu có, sau đó chạy container mới
                    // Sử dụng SSH Agent để kết nối SSH không cần mật khẩu
                    sshagent(credentials: ['ec2-key']) { // ID của SSH Credential trong Jenkins
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_PUBLIC_IP} <<EOF
                            docker pull ${IMAGE_NAME}:latest
                            docker stop ${APP_NAME} || true
                            docker rm ${APP_NAME} || true
                            docker run -d --name ${APP_NAME} -p 80:80 ${IMAGE_NAME}:latest
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Build process finished.'
            // Đăng xuất khỏi Docker Hub sau khi hoàn thành
            sh 'docker logout'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
        success {
            echo 'Pipeline successful!'
        }
    }
}
