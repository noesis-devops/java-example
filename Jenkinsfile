pipeline {
  agent {
    kubernetes {
      yaml '''
      apiVersion: v1
      kind: Pod
      spec:
        containers:
        - name: maven
          image: maven:3.8.6
          command:
          - sleep
          args:
          - infinity'''
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
            withSonarQubeEnv('sonarqube') {
              sh "${scannerHome}/bin/sonar-scanner -Dsonar.log.level=DEBUG -Dsonar.projectKey=java-example -Dsonar.verbose=true -Dsonar.exclusions=WebContent/vendor/*"
            }
          }
        }
      }
    }
    stage("Quality Gate") {
      steps {
        script {
          timeout(time: 1, unit: 'HOURS') { 
            def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            if (qg.status != 'OK') {
              error "Pipeline aborted due to quality gate failure: ${qg.status}"
            } else {
              println("quality gate passed!")
            }
          }
        }
      }
    }
  }
}
