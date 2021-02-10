def ecr = "https://063844947040.dkr.ecr.us-east-1.amazonaws.com"
def dockerLabel = "jenkins-slave-${UUID.randomUUID().toString()}"

pipeline {
  options {
    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '2', numToKeepStr: '10')
    disableConcurrentBuilds()
  }
  agent {
    kubernetes {
      label dockerLabel
      yaml """
apiVersion: v1
kind: Pod
spec:
  securityContext:
    fsGroup: 1000
  containers:
  - name: jnlp
    image: jenkinsci/jnlp-slave:latest
    tty: true
    resources:
      requests:
        memory: 128Mi
  - name: helm
    image:  alpine/helm:3.5.2
    tty: true
  - name: docker
    image: stmllr/docker-client:18.03
    tty: true
    command: 
    - cat
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock 
  serviceAccount: jenkins
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
"""
    }
  }
  stages {
    stage("Build docker image") {
      steps {
        script {
          container('docker') {
            docker.withRegistry(ecr, "ecr:us-east-1:ecr.cred") {
              sh """docker build --network="host" -t 063844947040.dkr.ecr.us-east-1.amazonaws.com/reali-test:latest .
              docker push 063844947040.dkr.ecr.us-east-1.amazonaws.com/reali-test:latest"""
            }
          }
        }
      }
    }
    stage("Deploy helm") {
      steps {
        script {
          container('helm') {
            sh """helm upgrade --install reali-test ./reali-test --namespace reali-test
            """
          }
        }
      }
    }
  }
}              
