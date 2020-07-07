pipeline {
  environment {
    registry = "anishnath/anchore"
    registryCredential = 'docker'
    dockerImage =  ''
}
agent any
stages {
  stage('Cloning Git') {
    steps {
      git 'https://github.com/anishnath/anchore.git'
 }
}
stage('Building image') {
  steps{
    script {
      dockerImage = docker.build registry + ":$BUILD_NUMBER"
      }
    }
  }
stage('Anchore Container Security Scan') {
  steps {
    //sh 'echo "docker.io/"anishnath/anchore" `pwd`/Dockerfile" > anchore_images'
    sh 'echo "anishnath/anchore" + ":$BUILD_NUMBER" ${WORKSPACE}/Dockerfile > anchore_images'
    anchore name: 'anchore_images'
    }
  }

stage('Deploy Image') {
  steps{
    script {
      docker.withRegistry( '', registryCredential ) {
      dockerImage.push()
      }
   }
 }
}
 
stage('Cleanup') {
  steps {
    sh'''
      for i in `cat anchore_images | awk '{print $1}'`;do docker rmi $i; done
 '''
      }
    }
  }
}
