pipeline {
  agent any
  environment {
    VERSION = """${sh(
            returnStdout: true,
            script: 'cat VERSION'
        )}""" 
        
    EXIT_STATUS = """${sh(
            returnStatus: true,
            script: 'exit 1'
        )}"""
  }
  
  stages {
    stage('Pre-Build') {
      steps {
        sh '''#!/bin/bash
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin 226347592148.dkr.ecr.ap-northeast-1.amazonaws.com
'''
      }
    }

    stage('Build') {
      steps {
        sh '''#!/bin/bash
chmod +x ./gradlew
./gradlew build
docker build -t 226347592148.dkr.ecr.ap-northeast-1.amazonaws.com/demo-backend:v1.1.0 . 
'''
      }
    }

    stage('Post-Build') {
      steps {
        sh 'docker push 226347592148.dkr.ecr.ap-northeast-1.amazonaws.com/demo-backend:v1.1.0'
      }
    }

    stage('Clone-helm-repo') {
      steps {
        git(url: 'https://github.com/dispiny/demo-charts', branch: 'master', credentialsId: '5edb4fde-dd7d-43d9-bcc4-d87afdc119c8')
      }
    }

    stage('helm-Build') {
      steps {
        sh '''#!/bin/bash
sed -i "s|version:.*|version: $VERSION|g" backend-skills-repo/Chart.yaml
sed -i "s|tag:.*|tag: v$VERSION|g" backend-skills-repo/values.yaml

'''
      }
    }

  }
}