pipeline {
    agent {
        kubernetes {
            label 'kaniko' // 파드 이름
            defaultContainer 'kaniko' // Kaniko 컨테이너 사용
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command: ['sleep']
    args: ['infinity']
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

    stages {
        stage('Checkout') {
            steps {
                // GitHub 리포지토리에서 코드 가져오기
                git 'https://github.com/gogoyooni/kaniko-test.git'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    container('kaniko') {
                        // Kaniko를 사용하여 Docker 이미지를 빌드하고 Docker Hub로 푸시
                        sh """
                        /kaniko/executor --dockerfile=Dockerfile --context=git://github.com/gogoyooni/kaniko-test.git --destination=taeyoondev/kaniko-test:v1.2 --cache=false --cleanup=true
                        """
                    }
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