pipeline {
    agent any

    environment {
        VENV_DIR     = 'venv'
        GCP_PROJECT  = 'hotelreservation-482911'
        GCLOUD_PATH  = '/var/jenkins_home/google-cloud-sdk/bin'
    }

    stages {

        stage('Cloning Github repo to Jenkins') {
            steps {
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

        stage('Setting up Virtual Environment and Installing Dependencies') {
            steps {
                echo 'Setting up Virtual Environment and Installing Dependencies............'
                sh '''
                    python --version
                    python -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -e .
                '''
            }
        }

        stage('Building and Pushing Docker Image to GCR') {
            steps {
                echo 'Building and Pushing Docker Image to GCR.............'
                withCredentials([
                    file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')
                ]) {
                    sh '''
                        export PATH=$PATH:${GCLOUD_PATH}

                        gcloud auth activate-service-account \
                            --key-file=${GOOGLE_APPLICATION_CREDENTIALS}

                        gcloud config set project ${GCP_PROJECT}
                        gcloud auth configure-docker --quiet

                        docker build -t gcr.io/${GCP_PROJECT}/ml-project:latest .
                        docker push gcr.io/${GCP_PROJECT}/ml-project:latest
                    '''
                }
            }
        }

    }
}


//         stage('Deploy to Google Cloud Run'){
//             steps{
//                 withCredentials([file(credentialsId: 'gcp-key' , variable : 'GOOGLE_APPLICATION_CREDENTIALS')]){
//                     script{
//                         echo 'Deploy to Google Cloud Run.............'
//                         sh '''
//                         export PATH=$PATH:${GCLOUD_PATH}


//                         gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}

//                         gcloud config set project ${GCP_PROJECT}

//                         gcloud run deploy ml-project \
//                             --image=gcr.io/${GCP_PROJECT}/ml-project:latest \
//                             --platform=managed \
//                             --region=us-central1 \
//                             --allow-unauthenticated
                            
//                         '''
//                     }
//                 }
//             }
//         }
        
//     }
// }