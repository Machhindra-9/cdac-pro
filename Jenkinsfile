pipeline {
    agent any
    environment {
        REPORT_DIR = "dependency-check-report"
        SONAR_HOST_URL = 'http://192.168.80.153:9000'
    }
    stages {
        stage('hello') {
            steps {
                echo 'hello'
            }
        }
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Machhindra-9/cdac-pro.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Retrieve the SonarQube Scanner tool (ensure the tool name matches your Global Tool Configuration)
                    def scannerHome = tool 'sonar'
                    
                    // Use the SonarQube environment that you configured in Jenkins
                    withSonarQubeEnv('sonar') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=devops-project-key \
                            -Dsonar.projectName="devops-project" \
                            -Dsonar.projectVersion=1.0 \
                            -Dsonar.sources=. \
                            -Dsonar.language=py \
                            -Dsonar.python.version=3 \
                            -Dsonar.host.url=${env.SONAR_HOST_URL}
                        """
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t first .'
                // Remove existing container if present
                script {
                    sh '''
                    CONTAINER_NAME=receipe
                    if [ $(docker ps -aq -f name=$CONTAINER_NAME) ]; then
                        docker stop $CONTAINER_NAME
                        docker rm $CONTAINER_NAME
                    fi
                    '''
                }
                // Remove dangling images
                sh 'docker images --filter "dangling=true" -q | xargs -r docker rmi'
            }
        }
        stage('Trivy Scan of Image') {
            steps {
                // Scan the Docker image and output the result in JSON format
                sh 'trivy image --format json --output trivy-report.json first'
            }
        }
        stage('Archive Trivy Report') {
            steps {
                // Archive the generated Trivy report so it can be viewed from the Jenkins build page
                archiveArtifacts artifacts: 'trivy-report.json', allowEmptyArchive: true
            }
        }
        stage('Remove Existing Container') {
            steps {
                script {
                    sh '''
                    CONTAINER_NAME=receipe
                    if [ $(docker ps -aq -f name=$CONTAINER_NAME) ]; then
                        docker stop $CONTAINER_NAME
                        docker rm $CONTAINER_NAME
                    fi
                    '''
                }
            }
        }
        stage('Deploy Container') {
            steps {
                sh 'docker container run --name receipe -d -p 8000:8000 first'
                echo 'Container deployed'
            }
        }
    }
}
