def kubernetes_config = """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: ci
    image: eclipseglsp/ci:0.0.3
    tty: true
    resources:
      limits:
        memory: "4Gi"
        cpu: "2"
      requests:
        memory: "4Gi"
        cpu: "2"
    command:
    - cat
    volumeMounts:
    - mountPath: "/home/jenkins"
      name: "jenkins-home"
      readOnly: false
    - mountPath: "/.yarn"
      name: "yarn-global"
      readOnly: false
  volumes:
  - name: "jenkins-home"
    emptyDir: {}
  - name: "yarn-global"
    emptyDir: {}
"""
pipeline {
    agent {
        kubernetes {
            label 'glsp-agent-pod'
            yaml kubernetes_config
        }
    }
    options {
        buildDiscarder logRotator(numToKeepStr: '15')
    }
    environment {
        YARN_CACHE_FOLDER = "${env.WORKSPACE}/yarn-cache"
        SPAWN_WRAP_SHIM_ROOT = "${env.WORKSPACE}"
        CHROME_BIN="/usr/bin/google-chrome"
    }
    stages {
        stage('Build client') {
            steps {
                container('ci') {
                    timeout(30){
                        dir('client') {
                            sh 'yarn install'
                        }
                    }
                }
            }
        }
        stage('Build server'){
            steps{
                container('ci'){
                    timeout(30){
                        dir('server'){
                            sh 'mvn clean verify --batch-mode package'
                        }
                    }
                }
            }
        }
        stage('E2E tests'){
            steps{
                container('ci'){
                    timeout(30){
                        dir('client'){
                            sh 'yarn e2etest'
                        }
                    }
                }
            }
        }
    }
}