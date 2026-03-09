pipeline {
    agent any
    
    parameters {
        string(name:'DOCKER_TAG', defaultValue:'latest', description: 'Docker tag')
    }
    
    tools {
        maven "maven3"
    }
    environment {
        SCANNER_HOME= tool "sonar-scanner"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/sachi3050/Multi-Tier-BankApp-CI.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('File System Scann') {
            steps {
                sh "trivy fs --format table -o fs.html . "
            }
        }
        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=GitOps-CI \
                      -Dsonar.projectKey=GitOps-CI \
                      -Dsonar.java.binaries=target '''
                }
            }
        }
        stage('Artifact Builds and Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'GitOps-Maven-Config-File', maven: 'maven3', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        stage('Docker Build and Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh 'docker build -t sachidananda06/gitops:${params.DOCKER_TAG} .'
                    }
                }
            }
        }
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o dimage.html sachidananda/gitops:${params.DOCKER_TAG} "
            }
        }
        stage('Publish to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh 'docker push sachidananda06/gitops:${params.DOCKER_TAG}'
                    }
                }
            }
        }
        stage('Update YAML Manifest in Other Repo') {
            steps {
                script {
                    withCredentials([gitUsernamePassword(credentialsId: 'git-cred', gitToolName: 'Default')]) {
                       sh ''' 
                       #Clone the Git Repo
                       git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/sachi3050/Multi-Tier-BankApp-CD.git' 
                       cd Multi-Tier-BankApp-CD
                       
                       #List file to confirm the Presence of Bankapp-ds.yml 
                       ls -l bankapp
                       
                       #Get the Absolute Path for the Current Directory
                       repo_dir=$(pwd)
                       
                       #Use the Absolute path for sed
                       sed -i 's|image: sachidananda06/gitops:.* | image: sachidananda06/gitops:'$(DOCKER_TAG)' |'
                       ${repo_dir}/bankapp/bankapp-ds.yml '''
                       
                       /// Confirm the Changes
                       sh '''
                          echo "UPDATED YAML file Content: "
                          cat Multi-Tier-BankApp-CD/bankapp/bankapp-ds.yml '''
                          
                       // configure the git for Committing the Changes and Pushing
                       sh '''
                       cd Multi-Tier-BankApp-CD  #Ensure you are in the Cloned Repo
                       git config user.email "snsbr2004@gmail.com"
                       git config user.name "GitOps-CI" '''
                       
                       //Commit and Push the Updated YAML file back to other Repository
                       sh '''
                       cd Multi-Tier-BankApp-CD
                       ls 
                       git add bankapp/bankapp-ds.yml
                       git commit -m "Update Image Tag to $(DOCKER_TAG) "
                       git push origin main
                       


                    }
                }
            }
        }
        stage('dummy') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
