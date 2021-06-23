pipeline{
    agent{
        label 'Slave'
    }

    tools{
        maven "m3"
    }
    environment{
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
    stages{
        stage('Clear running apps'){
            steps{
                sh 'docker rm -f pandaapp || true'
            }
        }
        stage('Get Code'){
            steps{
                git branch: 'pipeline', url: 'https://github.com/LucekC/panda_application.git'
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

    }
    post {
        always{
            sh 'docker stop pandaapp'
            deleteDir()
        }
    }    
}