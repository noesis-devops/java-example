// Uses Declarative syntax to run commands inside a container.
pipeline {
    agent {
        kubernetes {
            // Rather than inline YAML, in a multibranch Pipeline you could use: yamlFile 'jenkins-pod.yaml'
            // Or, to avoid YAML:
            // containerTemplate {
            //     name 'shell'
            //     image 'ubuntu'
            //     command 'sleep'
            //     args 'infinity'
            // }
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8.6
    resourceRequestMemory: '1000Mi'
    resourceLimitMemory: '2000Mi'
    command:
    - sleep
    args:
    - infinity
'''
            // Can also wrap individual steps:
            // container('shell') {
            //     sh 'hostname'
            // }
            defaultContainer 'maven'
        }
    }
    stages {
        stage('SCM') {
            steps {
    checkout scm
            }
  }
   stage('SonarQube analysis') {
       steps {
       script {
    def scannerHome = tool 'SonarScanner';
    nodejs(nodeJSInstallationName: 'node') {                  
      withSonarQubeEnv('sonarqube') { // If you have configured more than one global server connection, you can specify its name
        sh "${scannerHome}/bin/sonar-scanner -Dsonar.log.level=DEBUG -Dsonar.projectKey=java-example -Dsonar.verbose=true -Dsonar.javascript.node.maxspace=2000"
      }
    }
       }
       }
  }
  stage("Quality Gate"){
      steps {
       script {
  timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
    if (qg.status != 'OK') {
      error "Pipeline aborted due to quality gate failure: ${qg.status}"
    } else {
      println ("quality gate passed!")
    }
  }
       }
      }
}
    }
}
