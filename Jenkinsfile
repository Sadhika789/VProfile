pipeline {
    agent any
    tools{
        jdk 'JDK17'
        maven 'M3'
    }
    triggers{
        githubPush()
    }
        
    stages {
        stage('Checkout'){
            steps {
                git branch: 'master', url: 'https://github.com/Sadhika789/VProfile/'
            }
        }
        stage('package war'){
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }
        
            stage('build') {
               steps {
                  bat """
            minikube docker-env --shell=cmd > docker-env.bat
            call docker-env.bat
            cd %WORKSPACE%
            docker build -t localhost:5000/vprofile:%BUILD_NUMBER% -f %WORKSPACE%/Dockerfile %WORKSPACE%
            docker push localhost:5000/vprofile:%BUILD_NUMBER%
        """
            }
        }
        stage('Test Kubeconfig') {
          steps {
            bat "kubectl --kubeconfig=\"C:\\Program Files\\Jenkins\\.kube\\config\" get pods"
       }
    }
       stage('Update k8s manifests in github') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'github-creds',
                                          usernameVariable: 'GIT_USER',
                                          passwordVariable: 'GIT_TOKEN')]) {
            bat """
            powershell -Command "(Get-Content src/k8s/deployment.yml) -replace 'image:.*', 'image:localhost:5000/vprofile:%BUILD_NUMBER%' | Set-Content src/k8s/deployment.yml"
           
            git add src/k8s/deployment.yml
            git commit -m "Update image tag to %BUILD_NUMBER%"
            git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/Sadhika789/VProfile.git
            git push origin master
            """
        }
    }
}

         
}
}
