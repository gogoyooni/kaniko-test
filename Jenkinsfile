pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            label 'kube-agent'  // Pod Template에서 설정한 이름
            serviceAccount 'jenkins-admin'  // 기존에 생성한 서비스 계정 지정
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  - name: kubectl
    image: docker.io/bitnami/kubectl
    command:
    - sleep
    args:
    - 99d
    tty: true
    securityContext:
      runAsUser: 1000
  restartPolicy: Never
  volumes:
    - name: kaniko-secret
      secret:
        secretName: dockercred
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

                            cat ${WORKSPACE}/k8s/deployment.yaml | sed 's/\${TAG}/${DOCKER_TAG}/g' | kubectl apply -f -
                            
                            # deployment가 완전히 롤아웃될 때까지 대기
                            kubectl rollout status deployment/kaniko-test-app
                            
                            # 서비스 정보 출력
                            echo "Application deployed! Service details:"
                            kubectl get svc kaniko-test-service
                        """
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