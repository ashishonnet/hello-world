podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: maven
        image: maven:3.8.6-jdk-11
        command:
        - sleep
        args:
        - 99d
      - name: kubectl
        image: bitnami/kubectl:latest
        command:
        - sleep
        args:
        - 99d
        tty: true
        securityContext:
          runAsUser: 0
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        imagePullPolicy: Always
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: kaniko-secret2
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: kaniko-secret2
        secret:
          secretName: harborcred2
          items:
          - key: .dockerconfigjson
            path: config.json
''') {
  node(POD_LABEL) {
    stage('Get a Maven project') {
      git url: 'https://github.com/ashishonnet/hello-world.git', branch: 'main'
      container('maven') {
        stage('Build a Maven project') {
          sh '''
          mvn -B -DskipTests clean package
          '''
        }
        // stage('Scan a Maven project') {
        //   sh '''
        //   mvn sonar:sonar -Dsonar.host.url=http://172.29.66.98:32668/ -Dsonar.login=sqp_0ac91fd86b7ca14f9f04bc9131f3ae0912344db4 -Dsonar.projectKey=MEGH
        //   '''
        // }

      }
    }

    // stage('SonarQube Scan') {
    //   git url: 'https://github.com/ashishonnet/kubernetes-kaniko.git', branch: 'main'
    //   container('maven') {
    //     stage('Scan a Maven project') {
    //       sh '''
    //       mvn sonar:sonar -Dsonar.host.url=http://172.29.66.98:32668/ -Dsonar.login=sqp_0ac91fd86b7ca14f9f04bc9131f3ae0912344db4 -Dsonar.projectKey=MEGH
    //       '''
    //     }
    //   }
    // }

    stage('Create Image') {
      container('kaniko') {
        stage('Push Image to Container Repository') {
            
          sh '''
            /kaniko/executor --dockerfile `pwd`/Dockerfile \
            --verbosity debug \
            --context `pwd` \
            --skip-tls-verify \
            --insecure \
            --destination 172.29.66.98:30002/demo/hello-world:${BUILD_NUMBER} \
          '''
        }
      }
    }

    stage('Deploy') {
        container(name: 'kubectl', shell: '/bin/sh') {
         
                script {
                    withCredentials([file(credentialsId: 'TMPKUBECONFIG', variable: 'TMPKUBECONFIG_FILE')]) {
                    sh '''
                    cp \$TMPKUBECONFIG_FILE /.kube/config
                    cat deployment.yaml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" | kubectl apply -f -
                    '''
                }
}
        
        
}
}
  }
}

