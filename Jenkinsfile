pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes' // 이 부분은 Jenkins에서 정의된 Kubernetes 클라우드를 사용
            namespace 'devops-tools' // devops-tools 네임스페이스 내에서 파드를 생성
            label 'kubeagent' // PodTemplate에서 정의한 label과 일치시켜야 합니다.
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "300m"
      limits:
        memory: "512Mi"
        cpu: "500m"
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  volumes:
  - name: kaniko-secret
    secret:
      secretName: regcred
      items:
        - key: .dockerconfigjson
          path: config.json
"""
        }
    }
    
    stages {
        stage('Build and Push with Kaniko') {
            steps {
                container('kaniko') {
                    sh '''
                        /kaniko/executor \
                        --context=git://github.com/gogoyooni/kaniko-test.git \
                        --dockerfile=Dockerfile \
                        --destination=taeyoondev/kaniko-test:v1.1 \
                        --cache=false \
                        --cleanup=true
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
        }
    }
}
