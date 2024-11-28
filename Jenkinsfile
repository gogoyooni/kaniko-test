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

    parameters {
        string name: 'IMAGE_NAME', defaultValue: 'kaniko-test'
        string name: 'IMAGE_REGISTRY_ACCOUNT', defaultValue: 'taeyoondev'
    }

    stages {
        stage('Build and Tag Docker Image') {
            steps {
                container('kaniko') {
                sh """/kaniko/executor --dockerfile=Dockerfile --context=git://github.com/gogoyooni/kaniko-test.git --destination=${params.IMAGE_REGISTRY_ACCOUNT}/${params.IMAGE_NAME}:${env.BUILD_NUMBER} --cache=false --cleanup=true"""
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