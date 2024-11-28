pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
  namespace: default
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    args:
    - "--dockerfile=Dockerfile"
    - "--context=git://github.com/gogoyooni/kaniko-test.git" #git://github.com/kunchalavikram1427/connected-app.git#refs/heads/master
    - "--destination=taeyoondev/kaniko-test:1.1"
    - "--cache=false"
    - "--cleanup=true"
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
    - name: kaniko-secret
      secret:
        secretName: regcred  // Docker Hub 인증 정보
        items:
        - key: .dockerconfigjson
          path: config.json
"""
        }
    }

    environment {
        IMAGE_NAME = 'kaniko-test'
        IMAGE_REGISTRY_ACCOUNT = 'taeyoondev'
    }

    stages {
        stage('Build and Tag Docker Image') {
            steps {
                container(name:'kaniko', shell: '/busyboxy/sh') {
                sh """#!/busyboox/sh
                    echo "FROM jenkins/inbound-agent:latest" > Dockerfile
                    /kaniko/executor --context `pwd` --destination $IMAGE_REGISTRY_ACCOUNT/$IMAGE_NAME:$env.BUILD_NUMBER --cache false --cleanup true"""
                }
            }
        }

        stage("Clean up the workspace") {
            steps {
                script{
                    deleteDir(); // 현재 워크스페이스 정리
                    echo "Workspace cleaned up."
                }
            }
        }
    }
    


    
        
    
}