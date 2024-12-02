pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
  namespace: devops-tools
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  - name: kubectl
    image: docker.io/bitnami/kubectl
    command:
    - cat
    tty: true
    securityContext:
    runAsUser: 1000
  restartPolicy: Never
  volumes:
    - name: kaniko-secret
      secret:
        secretName: dockercred  // Docker Hub 인증 정보
        items:
        - key: .dockerconfigjson
          path: config.json
"""
        }
    }

    environment {
        DOCKER_IMAGE = "taeyoondev/kaniko-test"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Build and Tag Docker Image') {
            steps {
                container('kaniko') {
                    script {
                        sh '''
                            /kaniko/executor \
                            --context=${WORKSPACE} \
                            --dockerfile=${WORKSPACE}/Dockerfile \
                            --destination=${DOCKER_IMAGE}:${DOCKER_TAG}
                        '''
                    }
                }
            }
        }

        stage('Test Kubernetes Connection') {
            steps {
                container('kubectl') {
                    script {
                        sh """
                            echo "Testing kubectl connection..."
                            kubectl version --client
                            kubectl get pods
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