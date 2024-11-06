pipeline {
    agent any

    environment {
        // IDs of the credentials stored in Jenkins
        AWS_CREDENTIALS_ID = 'AWS_CREDENTIALS'
        GIT_CREDENTIALS_ID = 'GIT_SERVER_CREDENTIALS'
        // AWS credentials will be provided by the user at runtime
        AWS_ACCESS_KEY_ID = ''
        AWS_SECRET_ACCESS_KEY = ''
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://your-repo-url.git', branch: 'main'
            }
        }

        stage('Setup Environment') {
            steps {
                sh 'python3 -m venv venv'
                sh './venv/bin/pip install -r requirements.txt'
            }
        }

        stage('Start Application') {
            steps {
                sh 'nohup ./venv/bin/python app.py &'
            }
        }

        stage('Wait for User Input') {
            steps {
                script {
                    // Wait for the user to interact with the application
                    input message: 'Please validate AWS credentials and input VPC IDs and Region in the application UI.'
                }
            }
        }

        stage('Process Terraform Module') {
            steps {
                script {
                    // Assuming the application has generated the Terraform module in 'Child_Module'
                    // Push the module to a new repository
                    sh '''
                    # Generate a unique repository name
                    REPO_NAME="terraform-module-$(date +%s)"
                    
                    # Create a new repository (assuming GitHub and using GitHub CLI)
                    gh repo create your-org/$REPO_NAME --private --confirm
                    
                    # Initialize the new repository
                    cd Child_Module
                    git init
                    git remote add origin https://github.com/your-org/$REPO_NAME.git
                    git add .
                    git commit -m "Add generated Terraform module"
                    git push -u origin master
                    '''

                    // Store the repository URL to display later
                    env.GENERATED_REPO_URL = "https://github.com/your-org/${REPO_NAME}"
                }
            }
        }

        stage('Display Results') {
            steps {
                echo "The generated Terraform module is available at: ${env.GENERATED_REPO_URL}"
            }
        }
    }

    post {
        always {
            // Clean up: Kill the Flask application if necessary
            sh 'pkill -f app.py || true'
        }
    }
}
