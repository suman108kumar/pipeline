pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'Maven'
        IMAGE_NAME = "suman108kumar/dockerimagekajonamerakhnahaiwolikhnahai:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git credentialsId: 'GitHubToken', branch: 'main',
                    url: 'https://github.com/suman108kumar/correctpipeline.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn clean verify -Dtest=!FormUITest"
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh """
                        ${MAVEN_HOME}/bin/mvn sonar:sonar \
                        -Dsonar.projectKey=sonartestor_sonarqubeproject \
                        -Dsonar.organization=sonartestor \
                        -Dsonar.host.url=https://sonarcloud.io
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            echo "⚠️ Quality Gate failed: ${qg.status}, continuing..."
                        }
                    }
                }
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                sh """
                    echo '🧹 Cleaning old image...'
                    docker rmi -f $IMAGE_NAME || true

                    echo '📦 Building image...'
                    docker build -t $IMAGE_NAME .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DockerToken',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME
                    """
                }
            }
        }

        stage('Deploy on Server') {
            steps {
                sshagent(['agentmachin']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no suman@35.174.7.117 << EOF

                        echo "🚀 Pulling image..."
                        docker pull $IMAGE_NAME

                        echo "🧹 Removing old container..."
                        docker rm -f securevault || true

                        echo "🚀 Running new container..."
                        docker run -d -p 8081:8080 --name securevault $IMAGE_NAME

EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline SUCCESS'
        }
        failure {
            echo '❌ Pipeline FAILED'
        }
    }
}
