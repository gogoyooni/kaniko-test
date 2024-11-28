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

    parameters {
        string name: 'IMAGE_NAME', defaultValue: 'kaniko-test'
        string name: 'IMAGE_REGISTRY_ACCOUNT', defaultValue: 'taeyoondev'
    }

    stages {
        stage('Build and Tag Docker Image') {
            steps {
                container(name:'kaniko', shell: '/busyboxy/sh') {
                sh """#!/busyboox/sh
                    echo "FROM jenkins/inbound-agent:latest" > Dockerfile
                    /kaniko/executor --context `pwd` --destination ${params.IMAGE_REGISTRY_ACCOUNT}/${params.IMAGE_NAME}:${env.BUILD_NUMBER} --cache false --cleanup true"""
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