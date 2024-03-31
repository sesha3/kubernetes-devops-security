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
          securityContext:
            fsGroup: 1000
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
            envFrom:
              - configMapRef:
                  name: build-config
            volumeMounts:
              - mountPath: /home/user/.local/share/buildkit
                name: buildkitd
              - name: registry-creds
                mountPath: /home/user/.docker
                readOnly: true
          volumes:
            - name: buildkitd
              emptyDir: {}
            - name: registry-creds
              secret:
                secretName: registry-creds
                defaultMode: 0400
                items:
                  - key: .dockerconfigjson
                    path: config.json
        '''
    }
  }
  stages {
      // stage('Checkout') {
      //       steps {
      //         git branch: 'main',
      //             url: 'https://github.com/sesha3/kubernetes-devops-security.git'
      //       }
      //   }

      stage('Unit Tests') {
            steps {
              container('maven') {
                sh 'mvn test'
              }
            }
            post{
              always{
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }

      stage('Build Artifact - Maven') {
            steps {
              container('maven') {
                sh 'mvn clean package -DskipTests=true'
                archive 'target/*.jar'
              }
            }
        }

      stage('Build and Push Container Images') {
            steps {
              container('buildkitd') {
                sh 'printenv'
                sh 'echo GIT_COMMIT: $GIT_COMMIT' //added comment
                sh 'buildctl build --frontend=dockerfile.v0 --local context=. --local dockerfile=. --output type=image,name=${REGISTRY}/${REPOSITORY}:$BUILD_NUMBER,push=true'
              }
            }
        }
    }
}