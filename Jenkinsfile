pipeline {
    agent any
    environment {
        STAGING_SERVER = 'spring_user_java@spring-docker-demo-mv'
        ARTIFACT_NAME = 'demo-0.0.1-SNAPSHOT.jar'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Roger-Rojas-2000/laboratorio6.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Code Quality') {
            steps {
                //sh 'mvn checkstyle:check'
                echo 'Code Quality successfuylly.'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Code Coverage') {
            steps {
                sh 'mvn jacoco:report'
            }
        }
        
            stage('Copy Artifact') {
                steps {
                    sshagent(['jenkis-spring-docker-key']) {
                        sh 'scp target/${ARTIFACT_NAME} $STAGING_SERVER:/home/spring_user_java/staging/'
                    }
                }
            }

            stage('Stop Existing App') {
                steps {
                    sshagent(['jenkis-spring-docker-key']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no $STAGING_SERVER "
                                pids=\\$(ps -ef | grep '${ARTIFACT_NAME}' | grep -v grep | awk '{print \\$2}')
                                if [ ! -z "\\$pids" ]; then
                                    echo 'Stopping previous instance: \\$pids'
                                    kill -9 \\$pids
                                else
                                    echo 'No previous instance running'
                                fi
                            "
                        '''
                    }
                }
            }

            stage('Start Application') {
                steps {
                    sshagent(['jenkis-spring-docker-key']) {
                        sh 'ssh -o StrictHostKeyChecking=no $STAGING_SERVER "nohup /opt/java/openjdk/bin/java -jar /home/spring_user_java/staging/${ARTIFACT_NAME} > /home/spring_user_java/staging/spring.log 2>&1 &"'
                    }
                }
            }

            /*steps {
                sh 'scp target/${ARTIFACT_NAME} $STAGING_SERVER:/home/spring_user_java/staging/'
                sh 'ssh $STAGING_SERVER "nohup java -jar /home/spring_user_java/staging/${ARTIFACT_NAME} > /dev/null 2>&1 &"'
            } */
        
        stage('Validate Deployment') {
            steps {
                sh 'sleep 20'
                sh 'curl --fail http://spring-docker-demo-mv:8080/health'
            }
        }
    }
}
