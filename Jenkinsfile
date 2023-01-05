pipeline{
    parameters{
        choice(name: 'action', choices: 'create\ndestroy', description: 'here we can say whether to create or destroy the helm chart')
        string(name: 'reponame', defaultValue: 'bitnami', description: 'here you can define the repo name')
        string(name: 'repourl', defaultValue: 'https://charts.bitnami.com/bitnami', description: 'here you can define the repo url')
        string(name: 'chartname', defaultValue: 'tomcat', description: 'here you can define the chartname')
        string(name: 'releasename', defaultValue: 'my-release', description: 'here you can define the releasename')
    }
    agent any
    stages{
        stage('ssh connection'){
            steps{
                script{
                   sshagent(['awskeypair']) {
                     sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94'
                   }
                }
            }
        }
        stage('copy files'){
            steps{
                script{
                    sshagent(['awskeypair']) {
                        sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94'
                        sh 'scp -r /var/lib/jenkins/workspace/p3/* ec2-user@172.31.42.94:/home/ec2-user'
                    }
                }
            }
        }
        stage('install git and ansible'){
            steps{
                script{
                    sshagent(['awskeypair']) {
                        sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94'
                        sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94 sudo amazon-linux-extras install -y ansible2'
                        sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94 ansible-playbook /home/ec2-user/gitansible.yml'
                    }
                }
            }
        }
        stage('helm install'){
            steps{
                script{
                    sshagent(['awskeypair']) {
                        sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94'
                        sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94 ansible-playbook /home/ec2-user/helminstall.yml'
                    }
                }
            }
        }
        stage('apply helm chart'){
            when { expression { params.action == 'create'}}
            steps{
                script{
                    sshagent(['awskeypair']) {
                        def apply = false
                        try{
                            input message: "do you want to apply this ${chartname}" chart to the cluster, ok: "yes, apply"
                            apply = true
                        }
                        catch(err){
                            apply = false
                            currentBuild.result = 'ABORTED'
                        }
                        if(apply){
                            sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94'
                            sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94 helm repo add ${reponame} ${repourl}"
                            sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94 helm repo update ${reponame}"
                            sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94 helm install ${releasename} ${reponame}/${chartname}"
                        }
                    }
                }
            }
        }
        stage('delete helm chart'){
            when { expression { params.action == 'destroy'}}
            steps{
                script{
                    def apply = false
                    try{
                        input message: "do you want to delete this ${chartname} chart from the cluster", ok: 'yes, delete'
                        apply = true
                    }
                    catch(err){
                        apply = false
                        currentBuild.result = 'ABORTED'
                    }
                    if(apply){
                        sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94'
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.42.94 helm uninstall ${releasename}"
                    }
                }
            }
        }
    }
}