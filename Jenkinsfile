pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          annotations:
            container.apparmor.security.beta.kubernetes.io/buildkitd: unconfined
        spec:
          containers:
          - name: maven
            image: maven:alpine
            command:
            - cat
            tty: true
          - name: buildkitd
            image: moby/buildkit:master-rootless
            args:
              - --oci-worker-no-process-sandbox
            readinessProbe:
              exec:
                command:
                  - buildctl
                  - debug
                  - workers
              initialDelaySeconds: 5
              periodSeconds: 30
            livenessProbe:
              exec:
                command:
                  - buildctl
                  - debug
                  - workers
              initialDelaySeconds: 5
              periodSeconds: 30
            securityContext:
              seccompProfile:
                type: Unconfined
              runAsUser: 1000
              runAsGroup: 1000
            volumeMounts:
              - mountPath: /home/user/.local/share/buildkit
                name: buildkitd
          volumes:
            - name: buildkitd
              emptyDir: {}
        '''
    }
  }
  stages {
      stage('git version') {
            steps {
              sh "git version"
            }
        }

      stage('maven version') {
            steps {
              container('maven') {
                sh 'mvn -v' //test comment
              }
            }
        }

      stage('docker version') {
            steps {
              container('buildkitd') {
                sh 'buildctl -v'
              }
            }
        }
    }
}