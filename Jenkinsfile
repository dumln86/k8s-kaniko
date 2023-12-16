podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: maven
        image: maven:3.8.1-jdk-8
        command:
        - sleep
        args:
        - 99d
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''')
node(POD_LABEL) {
  stage('Get a Maven project') {
    git url: 'https://github.com/dumln86/k8s-kaniko.git', branch: 'main'
    {
      script
      {
        GIT_COMMIT = sh(
            script: 'git rev-parse -short=10 HEAD | tail -n +2',
            returnStdout: true
        )
        echo "Git committer email: ${GIT_COMMIT}"
      }
    }
    container('maven') {
      stage('Build a Maven project') {
        sh '''
        echo pwd
        '''
      }
    }
  }

  stage('Build Java Image') {
    container('kaniko') {
      stage('Build a Go project') {
        sh "/kaniko/executor --context `pwd` --destination dumln86/hello-kaniko:${GIT_COMMIT}"
      }
    }
  }
}

