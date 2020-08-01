pipeline {
  agent {
    docker {
      image 'node:12-alpine'
      args '-p 20001-20100:3000'
    }
  }
  environment {
    CI = 'true'
    HOME = '.'
  }
  stages {
    stage('Install Packages') {
      steps {
        sh 'npm install'
      }
    }
    stage('Test and Build') {
      parallel {
        stage('Run Tests') {
          steps {
            sh 'npm run test'
          }
        }
        stage('Create Build Artifacts') {
          steps {
            sh 'npm run build'
          }
        }
      }
    }
    stage('Deployment') {
      parallel {
        stage('Staging') {
          when {
            branch 'staging'
          }
          steps {
            withAWS(region:'ap-south-1',credentials:'Jenkins-Credential-ID-AWS') {
              s3Delete(bucket: 'jenkins-syam', path:'**/*')
              s3Upload(bucket: 'jenkins-syam', workingDir:'build', includePathPattern:'**/*');
            }
            mail(subject: 'Staging Build', body: 'New Deployment to Staging', to: 'syam.rs01@gmail.com')
          }
        }
        stage('Production') {
          when {
            branch 'master'
          }
          steps {
            withAWS(region:'ap-south-1',credentials:'Jenkins-Credential-ID-AWS') {
              s3Delete(bucket: 'jenkins-syam', path:'**/*')
              s3Upload(bucket: 'jenkins-syam', workingDir:'build', includePathPattern:'**/*');
            }
            mail(subject: 'Production Build', body: 'New Deployment to Production', to: 'syam.rs01@gmail.com')
          }
        }
      }
    }
  }
}