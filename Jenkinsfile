import java.text.SimpleDateFormat

pipeline {
  agent {
    label "test"
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: "2"))
    disableConcurrentBuilds()
  }
  stages {
    stage("build") {
      steps {
        script {
          def dateFormat = new SimpleDateFormat("yy.MM.dd")
          currentBuild.displayName = dateFormat.format(new Date()) + "-" + env.BUILD_NUMBER
        }
        git "https://github.com/vfarcic/go-demo-2.git"
        sh "docker image build -f Dockerfile.multistage -t vfarcic/go-demo-2 ."
      }
    }
    stage("release") {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "docker",
          usernameVariable: "USER",
          passwordVariable: "PASS"
        )]) {
          sh "docker login -u $USER -p $PASS"
        }
        sh "docker push vfarcic/go-demo-2"
        sh "docker image tag vfarcic/go-demo-2 vfarcic/go-demo-2:${currentBuild.displayName}"
        sh "docker push vfarcic/go-demo-2:${currentBuild.displayName}"
      }
    }
  }
}