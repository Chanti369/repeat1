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
    }
}