pipeline {
    agent any
    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'pet-set19',
                credentialsId: 'git-cred',
                url: 'https://github.com/CloudHight/usteam.git'
            }
        }
        stage('Code Analysis') {
            steps {
               withSonarQubeEnv('sonarqube') {
                  sh "mvn sonar:sonar"
               }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 2, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage('Build Artifact') {
            steps {
                sh 'mvn -f pom.xml clean package'
            }
        }
        stage('Push Artifact to Nexus Repo') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'spring-petclinic',
                classifier: '',
                file: 'target/spring-petclinic-2.4.2.war',
                type: 'war']],
                credentialsId: 'nexus-cred',
                groupId: 'Petclinic',
                nexusUrl: '13.37.228.208:8081',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: 'nexus-repo',
                version: '1.0'
            }
        }
        stage('Trigger Playbooks on Ansible') {
            steps {
                sshagent (['ansible-key']) {
                      sh 'ssh -t -t ec2-user@13.38.110.100 -o strictHostKeyChecking=no "cd /etc/ansible && ansible-playbook /opt/docker/docker-image.yml"'
                      sh 'ssh -t -t ec2-user@13.38.110.100 -o strictHostKeyChecking=no "cd /etc/ansible && ansible-playbook /opt/docker/docker-container.yml"'
                      sh 'ssh -t -t ec2-user@13.38.110.100 -o strictHostKeyChecking=no "cd /etc/ansible && ansible-playbook /opt/docker/newrelic-container.yml"'
                  }
              }
        }
        stage('slack notification') {
            steps {
                slackSend channel: '29th-april-jenkins-pipeline-project-eu-team',
                message: 'successful application deployment',
                tokenCredentialId: 'slack'
            }
        }
    }
}
