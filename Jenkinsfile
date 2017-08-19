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
    stage("checkout") {
      steps {
        script {
          def props = readProperties file: "/run/secrets/cluster-info.properties"
          env.HOST_IP = props.hostIp
          env.DOCKER_HUB_USER = props.dockerHubUser
        }
        checkout scm
        stash name: "compose", includes: "docker-compose.yml"
      }
    }
    stage("build") {
      steps {
        dockerBuild("go-demo-2", env.DOCKER_HUB_USER)
      }
    }
    stage("functional") {
      steps {
        dockerFunctional("go-demo-2", env.DOCKER_HUB_USER, env.HOST_IP, "/demo")
      }
    }
    stage("release") {
      when {
        branch "master"
      }
      steps {
        dockerRelease("go-demo-2", env.DOCKER_HUB_USER)
      }
    }
    stage("deploy") {
      when {
        branch "master"
      }
      agent {
        label "prod"
      }
      steps {
        unstash "compose"
        dockerDeploy("go-demo-2", env.DOCKER_HUB_USER, env.HOST_IP, "/demo")
      }
    }
  }
  post {
    always {
      dockerCleanup("go-demo-2")
    }
  }
}