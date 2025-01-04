pipeline {
  agent any
  stages {
    stage('Pre-Build') {
      steps {
        sh '''#!/bin/bash
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin 226347592148.dkr.ecr.ap-northeast-1.amazonaws.com
echo $VERSION
'''
      }
    }

    stage('Build') {
      steps {
        sh '''#!/bin/bash
chmod +x ./gradlew
./gradlew build
docker build -t 226347592148.dkr.ecr.ap-northeast-1.amazonaws.com/demo-backend:v$VERSION . 
'''
      }
    }

    stage('Post-Build') {
      steps {
        sh 'echo $VERSION'
        sh 'docker push 226347592148.dkr.ecr.ap-northeast-1.amazonaws.com/demo-backend:v$VERSION'
      }
    }

    stage('Clone-helm-repo') {
      steps {
        git(url: 'https://github.com/dispiny/demo-charts', branch: 'master', credentialsId: '5edb4fde-dd7d-43d9-bcc4-d87afdc119c8')
      }
    }

    stage('helm-Pre-Build') {
      steps {
        sh '''#!/bin/bash
echo $VERSION
sed -i "s|version:.*|version: $VERSION|g" backend-skills-repo/Chart.yaml
sed -i "s|tag:.*|tag: v$VERSION|g" backend-skills-repo/values.yaml

'''
      }
    }

    stage('helm-Build') {
      steps {
        sh '''#!/bin/bash
helm package backend-skills-repo
helm repo index . --merge index.yaml --url https://github.com/dispiny/demo-charts/releases/download/v$VERSION/'''
      }
    }

    stage('helm-Post-Build') {
      steps {
        sh '''#!/bin/bash
gh auth setup-git
gh release create v$VERSION backend-skills-repo-$VERSION.tgz -t v$VERSION --generate-notes
rm -rf *.tgz
git add -A 
git commit -m "$VERSION commit!"
git push origin master'''
      }
    }

  }
  environment {
    VERSION = """${sh(
                  returnStdout: true,
                  script: 'cat VERSION'
              )}"""
    }
  }