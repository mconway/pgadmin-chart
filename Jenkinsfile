pipeline {
  agent any

  options {
    timeout(time: 10, unit: 'MINUTES')
    ansiColor('xterm')
    checkoutToSubdirectory('pgadmin')
  }

  stages {
    stage('Checkout halkeye/helm-charts') {
      environment {
        GITHUB = credentials('github-halkeye')
      }
      steps {
        dir('helm-charts') {
          sh 'git clone -b gh-pages https://${GITHUB_USR}:${GITHUB_PSW}@github.com/halkeye/helm-charts.git .'
        }
      }
    }

    stage('Build') {
      steps {
        dir('pgadmin') {
          docker.image('dtzar/helm-kubectl').inside {
            sh 'helm init -c'
            sh 'helm lint .'
            sh 'helm package .'
            sh 'find'
          }
          stash(name:'helm-packages', allowEmpty: false, includes: '*.tgz')
        }
        dir('helm-charts') {
          dir('pgadmin') {
            unstash(name:'helm-packages')
            sh 'find'
          }
          sh 'find'
          docker.image('dtzar/helm-kubectl').inside {
            sh 'helm repo index ./'
          }
        }
      }
    }

    stage('Commit') {
      steps {
        dir('helm-charts') {
          sh 'find'
          sh 'git config --global user.email "jenkins@gavinmogan.com"'
          sh 'git config --global user.name "Jenkins"'
          sh 'git add .'
          sh 'git commit -m "Adding package"'
        }
      }
    }

    stage('Deploy') {
      when {
        branch 'master'
      }
      environment {
        GITHUB = credentials('github-halkeye')
      }
      steps {
        sh 'echo ${GITHUB_USR} pushing '
        sh 'git config --global user.email "jenkins@gavinmogan.com"'
        sh 'git config --global user.name "Jenkins"'
        sh 'git config --global push.default simple'
        sh 'git push https://${GITHUB_USR}:${GITHUB_PSW}@github.com/halkeye/helm-charts.git'
      }
    }
  }
}
