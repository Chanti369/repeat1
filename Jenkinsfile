pipeline{
    agent{
        node{
            label 'awsec2instance'
        }
    }
    tools{
        maven 'MAVEN'
    }
    stages{
        stage('git checkout'){
            steps{
                script{
                    git branch: 'main', url: 'https://github.com/Chanti369/repeat1.git'
                }
            }
        }
        stage('mvn install'){
            steps{
                script{
                    sh 'mvn clean install'
                }
            }
        }
        stage('static code analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube') {
                        sh 'mvn clean install sonar:sonar'
                    }
                }
            }
        }
        stage('quality gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube'
                }
            }
        }
        stage('nexus artifact uploader'){
            steps{
                script{
                    def readpom = readMavenPom file: 'pom.xml'
                    def readversion = readpom.version
                    def readrepo = readversion.endsWith("SNAPSHOT") ? "repeat1-snapshot" : "repeat1-release"
                    nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'nexus-creds', groupId: 'com.javatechie', nexusUrl: '15.207.110.19:8081', nexusVersion: 'nexus3', protocol: 'http', repository: readrepo, version: readversion
                }
            }
        }
        stage('docker build image'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
                        sh 'docker login -u $dockerusername -p $dockerpassword'
                        sh 'docker build -t $JOB_NAME:v1.$BUILD_ID .'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dockerusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dockerusername/$JOB_NAME:latest'
                        sh 'docker rmi -f $JOB_NAME:v1.$BUILD_ID'
                    }
                }
            }
        }
        stage('docker push'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
                        sh 'docker login -u $dockerusername -p $dockerpassword'
                        sh 'docker push $dockerusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker push $dockerusername/$JOB_NAME:latest'
                        sh 'docker rmi -f $dockerusername/$JOB_NAME:latest'
                        sh 'docker rmi -f $dockerusername/$JOB_NAME:v1.$BUILD_ID'

                    }
                }
            }
        }
        stage('ansible-playbook'){
            steps{
                script{
                   sh 'ansible-playbook ansible.yml' 
                }
            }
        }
        stage('k8s misconfigs'){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=6bc37af1-e6bb-41e2-886b-ee0adad572d7']) {
                            sh 'helm datree test myapp'
                        }
                    }
                }
            }
        }
    }
}