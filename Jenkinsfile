pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins/agent-type: kaniko
    namespace: devops-tools
spec:
  containers:
    - name: jnlp
      image: jenkins/inbound-agent:latest
      resources:
        requests:
          memory: "512Mi"
          cpu: "500m"
        limits:
          memory: "1024Mi"
          cpu: "1000m"
    - name: kaniko
      image: gcr.io/kaniko-project/executor:debug
      command:
        - /busybox/cat
      tty: true
      resources:
        requests:
          memory: "400Mi"
          cpu: "300m"
        limits:
          memory: "500Mi"
          cpu: "500m"
      volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker/
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

    environment {
        DOCKERHUB_USERNAME = "taeyoondev"
        IMAGE_NAME = "kaniko-test"
    }

    stages {
        stage("Build Docker Image & Push to Docker Hub") {
            steps {
                container("kaniko") {
                    script {
                        def context = "."
                        def dockerfile = "Dockerfile"
                        def image = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}:v1.2"

                        sh "/kaniko/executor --context ${context} --dockerfile ${dockerfile} --destination ${image}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "The process is completed."
        }
    }
}
