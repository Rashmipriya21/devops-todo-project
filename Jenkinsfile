pipeline{
    agent any
    environment{
        SONAR_HOME = tool "Sonar"
    }
    
    stages {
        stage('code') {
            steps {
               git url : "https://github.com/Rashmipriya21/devops-todo-project.git", branch: "master"
               echo 'code cloned successfully'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("Sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=nodtodo -Dsonar.projectKey=nodetodo -X"
                }
                echo 'Code scanned Successfully'
            }
        }
        stage("Sonar Quality Gates"){
            steps{
                timeout(time: 1, unit:"MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        // stage('OWASP') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
        //         depedencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //         echo 'Dependency scanned successfully'
        //     }
        // }
        stage('Build') {
            steps {
                sh "docker build -t node-app-test-new ."
                echo 'Image Build Successfully'
            }
        }
        stage('Trivy Scan') {
            steps {
                sh "trivy image node-app-test-new"
                echo 'Code looks good- scanned by trivy- Ready to Deploy'
            }
        }
        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId:"Docker-Hub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker tag node-app-test-new:latest ${env.dockerHubUser}/node-app-test-new:latest"
                    sh "docker push ${env.dockerHubUser}/node-app-test-new:latest"
                    echo 'Image Pushed Successfully'
                }
                
            }
        }
        stage('Deploy') {
            steps {
                sh "docker-compose down && docker-compose up -d"
                echo 'Application deploy Successfully'
            }
        }
    }
}
