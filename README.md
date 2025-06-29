```
pipeline {
    agent any

    environment {
        GIT_CREDENTIALS_ID = 'github-creds'
        PORT = '8090'
        TARGET_DIR = '/opt/collegeapp'
        PID_FILE = "${TARGET_DIR}/app.pid"
        LOG_FILE = "${TARGET_DIR}/app.log"
        JAR_NAME = 'College-Network-0.0.1-SNAPSHOT.jar'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/swatikaushal432/college_network.git', branch: 'main'
            }
        }

        stage('Build with Maven') {
            steps {
                // Build a runnable JAR and skip test execution
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Stop Previous Application') {
            steps {
                script {
                    echo "Stopping previous application if running..."
                    sh "sudo pkill -f ${JAR_NAME} || true"
                    sh "sudo rm -f ${PID_FILE} || true"
                }
            }
        }

        stage('Deploy New Changes') {
            steps {
                script {
                    echo "Deploying new version of ${JAR_NAME}"
                    sh """
                        sudo mkdir -p ${TARGET_DIR}
                        sudo rm -f ${TARGET_DIR}/${JAR_NAME}
                        sudo cp target/${JAR_NAME} ${TARGET_DIR}/
                        sudo chown -R jenkins:jenkins ${TARGET_DIR}
                        sudo -u jenkins bash -c \\
                          'nohup java -jar ${TARGET_DIR}/${JAR_NAME} --server.port=${PORT} > ${LOG_FILE} 2>&1 &'
                        echo \$! | sudo tee ${PID_FILE}
                    """
                }
            }
        }
    }
}

```
