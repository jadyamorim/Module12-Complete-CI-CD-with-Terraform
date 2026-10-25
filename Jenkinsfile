#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@master', retriever: modernSCM(
  [$class: 'GitSCMSource',
  remote: 'https://github.com/jadyamorim/Module8-jenkins-shared-library.git',
  credentialsId: 'github-credentials'
  ]
)

pipeline {   
  agent any
  tools {
    maven 'Maven'
  }
  environment {
    IMAGE_NAME = 'jadyamorim/demo-app:java-maven-2.0'
  }
  stages {
    stage("build app") {
      steps {
        script {
          echo 'building application jar...'
          buildJar()
        }
      }
    }
    stage("build image") {
      steps {
        script {
          echo 'building docker image...'
          buildImage(env.IMAGE_NAME)
          dockerLogin()
          dockerPush(env.IMAGE_NAME)
        }
      }
    }
    stage("provision server") {
      environment {
        AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
        TF_VAR_env_prefix = 'test'
      }
      steps {
        script {
          dir('terraform') {
            sh "terraform init"
            sh "terraform apply --auto-approve"
            EC2_PUBLIC_IP = sh(
              script: "terraform output ec2-public_ip",
              returnStdout: true
            ).trim()
          }
        }
      }
    }
    stage("deploy") {
      steps {
        script {
          echo 'deploying docker image to EC2...'
          
          def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
          def ec2Instance = "ec2-user@35.180.151.121"

          sshagent(['server-ssh-key']) {
            sh "scp -o server-cmds.sh ${ec2Instance}:/home/ec2-user"
            sh "scp -o docker-compose.yaml ${ec2Instance}:/home/ec2-user"
            sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
          }
        }
      }
    }               
  }
}
