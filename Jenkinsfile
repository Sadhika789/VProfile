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
                sh 'mvn clean package -DskipTests'
            }
        }
        
            stage('build') {
               steps {
                  sh '''
             kubectl run kaniko-build-${BUILD_NUMBER} \
                  --rm -i --restart=Never \
                  --image=gcr.io/kaniko-project/executor:latest \
                  --overrides='{
                    "apiVersion": "v1",
                    "spec": {
                      "containers": [{
                        "name": "kaniko",
                        "image": "gcr.io/kaniko-project/executor:latest",
                        "args": [
                          "--dockerfile=/workspace/Dockerfile",
                          "--context=/workspace",
                          "--destination=localhost:5000/vprofile:${BUILD_NUMBER}"
                        ],
                        "volumeMounts": [{
                          "name": "workspace",
                          "mountPath": "/workspace"
                        }]
                      }],
                      "volumes": [{
                        "name": "workspace",
                        "hostPath": {
                          "path": "${WORKSPACE}"
                        }
                      }]
                    }
                  }'
        '''
            }
        }
        stage('Test Kubeconfig') {
          steps {
            sh 'kubectl --kubeconfig=/home/jenkins/.kube/config get pods'
       }
    }
       stage('Update k8s manifests in github') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'github-creds',
                                          usernameVariable: 'GIT_USER',
                                          passwordVariable: 'GIT_TOKEN')]) {
           sh '''
            sed -i "s|image.*|image: localhost:5000/vprofile:${BUILD_NUMBER}|" src/k8s/deployment.yml
            git add src/k8s/deployment.yml
            git commit -m "Update image tag to %BUILD_NUMBER%"
            git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/Sadhika789/VProfile.git
            git push origin master
         '''
        }
    }
}

         
}
}
