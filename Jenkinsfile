pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key')        // AWS Access Key from Jenkins
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')        // AWS Secret Key from Jenkins
        GITHUB_PAT            = credentials('github-pat')            // GitHub PAT from Jenkins
        AWS_DEFAULT_REGION    = 'us-east-1'                          // AWS region
    }

    stages {
        stage('Clean Workspace') {
            steps {
                echo 'Cleaning workspace...'
                deleteDir()
            }
        }

        stage('Checkout Code from GitHub') {
            steps {
                script {
                    dir('terraform') {
                        git(
                            url: 'https://github.com/AnandJoy7/terra_auto_Jenkins.git',
                            branch: 'main',
                            credentialsId: 'github-credentials'
                        )
                    }
                }
            }
        }

        stage('Run Terraform Automation Script') {
            steps {
                dir('terraform') {
                    echo 'Running Terraform automation script...'
                    sh 'python3 terraform_automation.py'
                }
            }
        }

        stage('Commit and Push Changes to Dev Branch') {
            steps {
                script {
                    dir('terraform') {
                        echo 'Configuring Git...'
                        sh '''
                            git config user.name "AnandJoy7"
                            git config user.email "k.anad548@gmail.com"
                        '''

                        echo 'Checking out dev branch or creating it...'
                        sh '''
                            git checkout dev || git checkout -b dev
                            git pull origin dev || echo "Dev branch not found, proceeding with a new one."
                        '''

                        echo 'Adding and committing changes...'
                        sh '''
                            git add .
                            git commit -m "Updated Terraform configuration from Jenkins" || echo "Nothing to commit"
                        '''

                        echo 'Pushing changes to dev branch using PAT...'
                        sh '''
                            git push https://${GITHUB_PAT}@github.com/AnandJoy7/terra_auto_Jenkins.git dev
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Archiving Terraform plan output...'
            dir('terraform') {
                archiveArtifacts artifacts: 'terraform_plan_output.txt', allowEmptyArchive: true
            }
        }
        success {
            echo 'Terraform automation completed successfully and changes pushed to dev branch.'
        }
        failure {
            echo 'Terraform automation failed. Please check the logs.'
        }
    }
}
