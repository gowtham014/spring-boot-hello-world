pipeline{
  agent any
  parameters {
        choice(name: 'BRANCH_NAME', choices: ['main', 'dev', 'prod'], description: 'Select branch')
  }
  stages{
    stage('checkout'){
      steps{
        git branch:"${params.BRANCH_NAME}",url:'https://github.com/gowtham014/spring-boot-hello-world.git'
      }
    }
    stage ('sonar-scan') {
      steps{
        withSonarQubeEnv('sonarqube-server') {
          sh """
          mvn clean verify sonar:sonar \
            -Dsonar.projectKey=sprinboot-demo \
            -Dsonar.host.url=http://34.57.57.2:9000 \
            -Dsonar.login=sqp_18f7cc916bcecc21d940c105864ecae99606f6a0
          """
        }
      }
    }
    stage('Quality Gate') {
      steps {
        script {
            // waitForQualityGate abortPipeline: true
          timeout(time: 5, unit: 'MINUTES') {
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status != 'OK') {
                            error "Quality Gate failed. Stopping the pipeline!"
            }
          }
        }
      }
    }
    stage ('build') {
      steps{
        sh "mvn clean package"
      }
    }
    stage('Upload to Nexus') {
      steps {
        nexusArtifactUploader(
          nexusVersion: 'nexus3',
          protocol: 'http',
          nexusUrl: '34.44.201.119:8081',
          groupId: 'DEV',
          version: "${env.BUILD_ID}",
          repository: 'spring',
          credentialsId: 'nexus-credentials',
          artifacts: [
            [
              artifactId: 'hello',
              classifier: '',
              file: 'target/spring-boot-2-hello-world-1.0.2-SNAPSHOT.jar',
              type: 'jar'
            ]
           
          ]
        )
      }
    }
    stage('allure-reports') {
      steps{
        script{
          allure([
            includeProperties: false,
            jdk: '',
            properties: [],
            reportBuildPolicy: 'ALWAYS',
            results: [[path: 'target/surefire-reports']]
            // tool: 'allure'
          ])
        }
      }
    }
    stage('vulnerability-scan') {
        steps{
            sh "grype dir:target/"
        }
    }
  }
  post {
    always {
      cleanWs() 
    }
  }
}