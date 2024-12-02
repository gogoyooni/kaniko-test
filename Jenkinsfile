pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            inheritFrom 'kube-agent'  // Pod Template에서 설정한 이름
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

                        # deployment.yaml에서 namespace 추출
                        echo "Extracting namespace from deployment.yaml..."
                        NAMESPACE=\$(cat \${WORKSPACE}/k8s/deployment.yaml | grep 'namespace:' | head -n1 | awk '{print \$2}')
                        echo "Detected namespace: \${NAMESPACE}"

                        # namespace 존재 여부 확인
                        echo "Checking namespace..."
                        if kubectl get namespace \${NAMESPACE} > /dev/null 2>&1; then
                            echo "Namespace \${NAMESPACE} already exists"
                        else
                            echo "Creating namespace \${NAMESPACE}..."
                            kubectl create namespace \${NAMESPACE}
                        fi

                        # deployment.yaml의 TAG 변수 치환 및 적용
                        echo "Applying deployment configuration..."
                        cat \${WORKSPACE}/k8s/deployment.yaml | sed 's/\\\${TAG}/${DOCKER_TAG}/g' | kubectl apply -f -

                        # deployment 존재 여부 확인 및 롤아웃 대기
                        echo "Checking deployment status..."
                        if kubectl get deployment kaniko-test-app -n \${NAMESPACE} > /dev/null 2>&1; then
                            echo "Deployment exists. Waiting for rollout..."
                            kubectl rollout status deployment/kaniko-test-app -n \${NAMESPACE}
                        else
                            echo "Deployment doesn't exist yet. Creating and waiting for rollout..."
                            cat \${WORKSPACE}/k8s/deployment.yaml | sed 's/\\\${TAG}/${DOCKER_TAG}/g' | kubectl apply -f -
                            kubectl rollout status deployment/kaniko-test-app -n \${NAMESPACE}
                        fi

                        # 서비스 존재 여부 확인 및 정보 출력
                        echo "Checking service status..."
                        if kubectl get svc kaniko-test-service -n \${NAMESPACE} > /dev/null 2>&1; then
                            echo "Service exists. Details:"
                            kubectl get svc kaniko-test-service -n \${NAMESPACE}
                        else
                            echo "Service doesn't exist yet. Creating..."
                            cat \${WORKSPACE}/k8s/deployment.yaml | sed 's/\\\${TAG}/${DOCKER_TAG}/g' | kubectl apply -f -
                            echo "Service created. Details:"
                            kubectl get svc kaniko-test-service -n \${NAMESPACE}
                        fi
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