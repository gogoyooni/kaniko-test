pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            namespace 'devops-tools'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: regcred
        mountPath: /kaniko/.docker
  volumes:
    - name: regcred
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
                        --context=dir://. \
                        --dockerfile=Dockerfile \
                        --destination=taeyoondev/kaniko-test:v1.2
                    '''
                }
            }
        }
    }
}
