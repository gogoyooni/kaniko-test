podTemplate(
    namespace: 'devops-tools'
    containers: [
        containerTemplate(
            name: 'kaniko',
            image: 'gcr.io/kaniko-project/executor:debug',
            command: 'sleep',
            args: '99d'
        )
    ],
    volumes: [
        secretVolume(
            mountPath: '/kaniko/.docker',
            secretName: 'regcred'
        )
    ]
) {
    node(POD_LABEL) {
        stage('Git Clone') {
            // GitHub 레포지토리 클론
            git branch: 'main',
                url: 'https://github.com/gogoyooni/kaniko-test.git'
        }
        
        stage('Build & Push Docker Image') {
            container('kaniko') {
                script {
                    // Kaniko를 사용하여 도커 이미지 빌드 및 푸시
                    sh '''
                        /kaniko/executor \
                        --context=dir://. \
                        --dockerfile=Dockerfile \
                        --destination=taeyoondev/kaniko-test:${BUILD_NUMBER} \
                    '''
                }
            }
        }
    }
}
