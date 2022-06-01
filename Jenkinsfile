pipeline {
  agent any

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and JaCoCo') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
      post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
      }
    }

    stage('Sonarqube') {
      steps {
        sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://aml-devsecops.eastus.cloudapp.azure.com:9000 -Dsonar.login=f5ab8c16710a06c7e020a4ebe5190d0282601317"
      }
      timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
    }
  }

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "dockerhub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t alorete76/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push alorete76/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

     stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 'alorete76/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          //sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    } 
    
  }

}