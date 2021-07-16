pipeline{
    agent{
        label 'Slave'
    }

    tools{
        maven "m3"
        terraform 'Terraform'
    }
    environment{
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
        ANSIBLE = tool name: 'Ansible', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
    }
    stages{
        stage('Clear running apps'){
            steps{
                sh 'docker rm -f pandaapp || true'
            }
        }
        stage('Get Code'){
            steps{
                git branch: 'infrastructure', url: 'https://github.com/LucekC/panda_application.git'
            }
        }
        stage('Build and Junit'){
            steps{
                sh 'mvn clean install'
            }
        }
        stage('Build Docker image'){
            steps{
                sh 'mvn package -Pdocker -Dmaven.test.skip=true'
            }
        }
        stage('Run Docker app'){
            steps{
                sh 'docker run -d -p 0.0.0.0:8080:8080 --name pandaapp -t ${IMAGE}:${VERSION}'
            }
        }
        stage('Test Selenium'){
            steps{
                sh 'mvn test -Pselenium'
            }
        }
        stage('Deploy jar to artifactory'){
            steps{
                configFileProvider([configFile(fileId: 'fa950e11-149a-4312-95c1-ce4cb45621fc', variable: 'mavensettings')]) {
                    // some block
                    sh "mvn -s $mavensettings deploy -Dmaven.test.skip=true -e"
                }
            }
        }
        stage('Run terraform') {
            steps {
                dir('infrastructure/terraform') { 
                sh 'terraform init && terraform apply -var-file panda.tfvars -auto-approve'
                } 
            }
        }
        stage('Copy Ansible role') {
            steps {
                sh 'cp -r infrastructure/ansible/panda/ /etc/ansible/roles/'
                sh 'ls -al /etc/ansible/roles/'
            }
        }
        stage('Run Ansible') {
            steps {
                dir('infrastructure/ansible') { 
                sh 'chmod 600 ../panda.pem'
                sh 'ansible-playbook -i ./inventory playbook.yml'
                } 
            }
        }

    }
    post {
        always{
            sh 'docker stop pandaapp'
            deleteDir()
        }
    }    
}