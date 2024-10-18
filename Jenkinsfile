pipeline {
    agent any 

    environment {
        // Define AWS and GitHub credentials in Jenkins
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key')   // AWS access key ID from Jenkins credentials
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')   // AWS secret access key from Jenkins credentials
        GITHUB_CREDENTIALS    = credentials('github-credentials') // GitHub credentials from Jenkins
        AWS_DEFAULT_REGION    = 'us-east-1' // Change to your desired AWS region
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    // Clone the GitHub repository into the 'terraform' directory
                    dir('terraform') {
                        git(
                            url: 'https://github.com/AnandJoy7/terra_auto.git',
                            branch: 'main',
                            credentialsId: 'github-credentials'
                        )
                    }
                }
            }
        }

        stage('Install Python Requirements') {
            steps {
                // Install the required Python packages from the 'terraform' directory
                dir('terraform') {
                    sh 'python3 -m pip install -r requirements.txt'
                }
            }
        }

        stage('Run Terraform Script') {
            steps {
                // Run the Python Terraform automation script from the 'terraform' directory
                dir('terraform') {
                    sh 'python3 terraform_automation.py'
                }
            }
        }

        stage('Push Updated Files to GitHub (dev branch)') {
            steps {
                script {
                    dir('terraform') {
                        // Configure Git with username and email
                        sh '''
                            git config user.name "AnandJoy7"
                            git config user.email "k.anad548@gmai.com.com"
                        '''

                        // Pull latest changes from dev branch to avoid conflicts
                        sh 'git checkout dev'
                        sh 'git pull origin dev'

                        // Add, commit, and push updated files
                        sh '''
                            git add .
                            git commit -m "Updated Terraform configuration from Jenkins automation" || echo "Nothing to commit"
                            git push origin dev
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            // Archive Terraform plan output for later reference, if it exists
            dir('terraform') {
                archiveArtifacts artifacts: 'terraform_plan_output.txt', allowEmptyArchive: true
            }
        }
        success {
            echo 'Terraform script executed and changes pushed to GitHub successfully.'
        }
        failure {
            echo 'Pipeline failed. Please check the logs for more details.'
        }
    }
}
