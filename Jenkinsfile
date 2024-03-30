pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: maven
            image: maven:alpine
            command:
            - cat
            tty: true
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
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
                sh 'mvn -v'
              }
            }
        }

      stage('docker version') {
            steps {
              container('maven') {
                sh 'docker -v'
                sh 'echo test'
              }
            }
        }

      stage('k8s version') {
            steps {
              sh "kubectl version --short"
            }
        }
    }
}