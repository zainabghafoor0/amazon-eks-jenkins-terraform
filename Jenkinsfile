pipeline {
  agent any

  triggers {
    // every 5 minutes (much safer than every minute)
    pollSCM('H/5 * * * *')
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Application') {
      steps {
        echo '=== Building Petclinic Application ==='
        sh 'mvn -B -DskipTests clean package'
      }
    }

    stage('Test Application') {
      steps {
        echo '=== Testing Petclinic Application ==='
        sh 'mvn test'
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
        }
      }
    }

    stage('Build Docker Image') {
      when { branch 'master' }
      steps {
        echo '=== Building Petclinic Docker Image ==='
        script {
          app = docker.build("zainabg12/petclinic-spinnaker-jenkins")
        }
      }
    }

    stage('Push Docker Image') {
      when { branch 'master' }
      steps {
        echo '=== Pushing Petclinic Docker Image ==='
        script {
          // Save commit hash + short tag to env so later stages can use it
          env.GIT_COMMIT_HASH = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
          env.SHORT_COMMIT = env.GIT_COMMIT_HASH.take(8)

          docker.withRegistry('https://registry.hub.docker.com', 'dockerHubCredentials') {
            app.push(env.SHORT_COMMIT)
            app.push("latest")
          }
        }
      }
    }

    stage('Remove local images') {
      steps {
        echo '=== Delete the local docker images ==='
        // Use your image name (zainabg12), not ibuchh
        sh("docker rmi -f zainabg12/petclinic-spinnaker-jenkins:latest || true")
        sh("docker rmi -f zainabg12/petclinic-spinnaker-jenkins:${env.SHORT_COMMIT} || true")
      }
    }
  }
}
