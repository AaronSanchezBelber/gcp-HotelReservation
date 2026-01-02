pipeline {
    agent any

    environment {
        VENV_DIR    = 'venv'                                // Python virtual environment
        GCP_PROJECT = 'hotelreservation-482911'             // GCP Project ID
        GCLOUD_PATH = '/var/jenkins_home/google-cloud-sdk/bin' // gcloud path
    }

    stages {

        // =========================
        // ========= CI ============
        // =========================

        stage('CI - Clone GitHub Repository') {
            steps {
                // Clone the GitHub repository
                echo 'Cloning Github repo to Jenkins............'
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'github-token',
                        url: 'https://github.com/AaronSanchezBelber/gcp-HotelReservation.git'
                    ]]
                )
            }
        }

        stage('CI - Setup Python & Install Dependencies') {
            steps {
                // Create virtual environment and install dependencies
                echo 'Setting up Python virtual environment............'
                sh '''
                    # Check Python version
                    python --version

                    # Create virtual environment
                    python -m venv ${VENV_DIR}

                    # Activate virtual environment
                    . ${VENV_DIR}/bin/activate

                    # Upgrade pip
                    pip install --upgrade pip

                    # Install project dependencies
                    pip install -e .
                '''
            }
        }

        stage('CI - Build & Push Docker Image to GCR') {
            steps {
                // Build and push Docker image
                echo 'Building and pushing Docker image to GCR.............'
                withCredentials([
                    file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')
                ]) {
                    sh '''
                        # Add gcloud to PATH
                        export PATH=$PATH:${GCLOUD_PATH}

                        # Authenticate with GCP
                        gcloud auth activate-service-account \
                            --key-file=${GOOGLE_APPLICATION_CREDENTIALS}

                        # Set GCP project
                        gcloud config set project ${GCP_PROJECT}

                        # Configure Docker authentication
                        gcloud auth configure-docker --quiet

                        # Build Docker image
                        docker build -t gcr.io/${GCP_PROJECT}/ml-project:latest .

                        # Push Docker image
                        docker push gcr.io/${GCP_PROJECT}/ml-project:latest
                    '''
                }
            }
        }

        // =========================
        // ========= CD ============
        // =========================

        stage('CD - Deploy to Google Cloud Run') {
            steps {
                // Deploy application to Cloud Run
                echo 'Deploying to Google Cloud Run.............'
                withCredentials([
                    file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')
                ]) {
                    sh '''
                        # Add gcloud to PATH
                        export PATH=$PATH:${GCLOUD_PATH}

                        # Authenticate with GCP
                        gcloud auth activate-service-account \
                            --key-file=${GOOGLE_APPLICATION_CREDENTIALS}

                        # Set GCP project
                        gcloud config set project ${GCP_PROJECT}

                        # Deploy to Cloud Run
                        gcloud run deploy ml-project \
                            --image=gcr.io/${GCP_PROJECT}/ml-project:latest \
                            --platform=managed \
                            --region=us-central1 \
                            --allow-unauthenticated
                    '''
                }
            }
        }

    }
}
