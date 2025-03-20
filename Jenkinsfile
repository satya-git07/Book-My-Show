pipeline {
    agent any

    environment {
        CLUSTER_NAME = 'my-cluster1'
        ZONE = 'us-west3'
        PROJECT_ID = 'lyrical-bus-452711-c5'
        GIT_BRANCH = 'main'
        //SONARQUBE_HOST = 'http://34.27.61.33:9000'  // Your SonarQube Server URL
        //SONARQUBE_PROJECT_KEY = 'netflix'  // Your SonarQube Project Key
        //SONARQUBE_TOKEN = 'sqp_03348e71430a3b7a3896ba9d3fd6fb2ce8cea3f9'  // Your SonarQube Token
    }

    stages {
        stage('Git Checkout') {
            steps {
                // Checkout the repository using the defined branch
                git url: 'https://github.com/satya-git07/Book-My-Show.git', branch: "${GIT_BRANCH}"
            }
        }
        
               // SonarQube Analysis Stage using sonar-scanner
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis using sonar-scanner
                    sh """
                        /opt/sonar-scanner-new/sonar-scanner-4.8.0.2856-linux/bin/sonar-scanner \
                            -Dsonar.projectKey="bookmyshow" \
                            -Dsonar.sources="." \
                            -Dsonar.host.url="http://192.168.2.179:9000" \
                            -Dsonar.login="squ_a30df14deb0651be3da03b112b282b7bcf1c8535"
                    """
                }
            }
        }

        
        stage("Docker Build & Push") {
            steps { 
                script {
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {   
                        sh "docker build -t bookmyshow ."
                        sh "docker tag bookmyshow:latest satyadockerhub07/bookmyshow:tagname"
                        sh "docker push satyadockerhub07/bookmyshow:tagname"
                    }
                }
            }
        }
        
          stage("TRIVY"){
                    steps{
                        sh "trivy image --timeout 20m satyadockerhub07/bookmyshow:tagname" 
                    }
                }

        
        stage('Terraform Init') {
            steps {
                // Initialize Terraform
                sh 'terraform init'
                sh 'ls -la'
            }
        }

        stage('Terraform Apply') {
            steps {
                // Authenticate and apply Terraform changes
                withCredentials([file(credentialsId: 'gcp-sa', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                // Authenticate and deploy to GKE
                withCredentials([file(credentialsId: 'gcp-sa', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                        sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${ZONE} --project ${PROJECT_ID}"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
