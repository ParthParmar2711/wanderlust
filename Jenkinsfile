pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
    }
    stages {
        stage("Clone Code from GitHub") {
            steps {
                git url: "https://github.com/ParthParmar2711/wanderlust", branch: "devops"
            }
        }
        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh """
                        $SONAR_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=wanderlust \
                        -Dsonar.projectKey=wanderlust
                    """
                }
            }
        }
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage("Deploy using Docker Compose") {
            steps {
                // Step 1: Stop and remove all running containers
            sh """
                echo "Stopping and removing all running containers..."
                docker ps -q | xargs -r docker stop || true
                docker ps -aq | xargs -r docker rm || true
            """

            // Step 2: Remove any existing Docker Compose setup
            sh """
                echo "Bringing down Docker Compose environment..."
                docker-compose down || true
                docker-compose rm -f || true
            """

            // Step 3: Start fresh containers
            sh """
                echo "Starting fresh containers with Docker Compose..."
                docker-compose up -d
            """

            }
        }
    }
}

   
