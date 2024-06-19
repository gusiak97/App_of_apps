def frontendImage="palczak/frontend"
def backendImage="palczak/backend"

pipeline {
    agent {
        label 'agent'
    }
    tools {
        terraform 'Terraform'
    }

    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
    }
    
    parameters {
        string(name: 'backendDockerTag', defaultValue: 'latest', description: 'Backend docker image tag')
        string(name: 'frontendDockerTag', defaultValue: 'latest', description: 'Frontend docker image tag')
    }

    stages {
        stage('Get Code') {
            steps {
                checkout scm // Get some code from a GitHub repository
            }
        }

        stage('Show version') {
            steps {
                script{
                    currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
                }
            }
        }

        stage('Clean running containers') {
            steps {
                sh "docker rm -f frontend backend"
            }
        }

        stage('Deploy application') {
            steps {
                script {
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", 
                             "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                        sh "docker-compose up -d"
                    }
                }
            }
        }

        //stage('Selenium tests') {
            //steps {
                //sh "pip3 install -r test/selenium/requirements.txt"
                //sh "python3 -m pytest test/selenium/frontendTest.py"
            //}
        //}

        stage('Run terraform') {
            steps {
                dir('Terraform') {                
                    git branch: 'main', url: 'https://github.com/palczak/terraform'
                    withAWS(credentials:'AWS', region: 'us-east-1') {
                            sh 'terraform init -backend-config=bucket=agnieszka-palczak-panda-devops-core-18'
                            sh 'terraform apply -auto-approve -var bucket_name=agnieszka-palczak-panda-devops-core-18'
                            
                    } 
                }
            }
        }
        stage('Run Ansible') {
            steps {
                script {
                    sh "pip3 install -r requirements.txt"
                    sh "ansible-galaxy install -r requirements.yml"
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", 
                            "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                            ansiblePlaybook inventory: 'inventory', playbook: 'playbook.yml'
                    }
                }
            }
        }
    }

    post {
        always {
          sh "docker-compose down"
          cleanWs()
        }
    }
}
