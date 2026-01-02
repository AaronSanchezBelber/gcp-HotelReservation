pipeline {
    agent any

    environment {
        VENV_DIR     = 'venv'                               # Python virtual environment
        GCP_PROJECT  = 'hotelreservation-482911'            # GCP Project ID
        GCLOUD_PATH  = '/var/jenkins_home/google-cloud-sdk/bin'  # gcloud binary path
    }

    stages {

        // =========================
        // ======== CI PART ========
        // =========================

        stage('CI - Clone GitHub Repository') {
            steps {
                # Clone the repository from GitHub
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

        stage('CI - Setup Python Environment & Install Dependencies') {
            steps {
                # Create Python virtual environment and install dependencies
                echo 'Setting up Virtual Environment and Installing Dependencies............'
                sh '''
                    python --version                         # Check Python version
                    python -m venv ${VENV_DIR}               # Create venv
                    . ${VENV_DIR}/bin/activate               # Activate venv
                    pip install --upgrade pip                # Upgrade pip
                    pip install -e .                         # Install project dependencies
                '''
            }
        }

        stage('CI - Build & Push Docker Image to GCR') {
            steps {
                # Authenticate to GCP and push Docker image
                echo 'Building and Pushing Docker Image to GCR.............'
                withCredentials([
                    file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')
                ]) {
                    sh '''
                        export PATH=$PATH:${GCLOUD_PATH}     # Add gcloud to PATH

                        gcloud auth activate-service-account \
                            --key-file=${GOOGLE_APPLICATION_CREDENTIALS}  # GCP auth

                        gcloud config set project ${GCP_PROJECT}            # Set project
                        gcloud auth configure-docker --quiet               # Docker auth

                        docker build -t gcr.io/${GCP_PROJECT}/ml-project:latest .  # Build image
                        docker push gcr.io/${GCP_PROJECT}/ml-project:latest        # Push image
                    '''
                }
            }
        }

        // =========================
        // ======== CD PART ========
        // =========================

        stage('CD - Deploy to Google Cloud Run') {
            steps {
                # Deploy the Docker image to Cloud Run
                echo 'Deploying to Google Cloud Run.............'
                withCredentials([
                    file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')
                ]) {
                    sh '''
                        export PATH=$PATH:${GCLOUD_PATH}     # Add gcloud to PATH

                        gcloud auth activate-service-account \
                            --key-file=${GOOGLE_APPLICATION_CREDENTIALS}  # GCP auth

                        gcloud config set project ${GCP_PROJECT}            # Set project

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
