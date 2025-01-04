pipeline {
  agent any
  stages {
    stage('Pre-Build') {
      steps {
        sh '''#!/bin/bash
env
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
        sh '''#!/bin/bash
rm -rf *
rm -rf .*'''
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
        sh '''
          git config user.name "dispiny"
          git config user.email "aws.pjm1024cl@gmail.com"
        '''

        withCredentials([usernamePassword(credentialsId: '5edb4fde-dd7d-43d9-bcc4-d87afdc119c8', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
          sh """
            git remote set-url origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/dispiny/demo-charts.git
            echo $GIT_PASSWORD | gh auth login --with-token 
          """
        }

        sh '''
NAME=$(gh release view v$VERSION --json assets --jq '.assets[].name' || echo nope)
isFrontend=$(echo $NAME | grep frontend | wc -l)
isBackend=$(echo $NAME | grep backend | wc -l)

if [ $isBackend -eq 0 ] && [ $isFrontend -eq 1 ]; then
  gh release upload v$VERSION backend-skills-repo-$VERSION.tgz
elif [ $isBackend -eq 1 ] && [ $isFrontend -eq 0 ]; then
  echo "Only Backend"
elif [ $isBackend -eq 0 ] && [ $isFrontend -eq 0 ]; then
  gh release create v$VERSION backend-skills-repo-$VERSION.tgz -t v$VERSION --generate-notes
elif [ $isBackend -eq 1 ] && [ $isFrontend -eq 1 ]; then
  echo "Full"
fi'''


        sh '''#!/bin/bash
            rm -rf *.tgz
            git add -A 
            git commit -m "$VERSION commit!"
            git push origin master
            rm -rf *
            rm -rf .*'''
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