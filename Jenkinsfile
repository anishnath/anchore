node {
  def app
  def dockerfile
  def anchorefile
  def repotag
  
  environment {
    registry = "anishnath/anchore"
    registryCredential = 'docker'
    dockerImage =  ''
   }

  try {
    stage('Checkout') {
      // Clone the git repository
      checkout scm
      def path = sh returnStdout: true, script: "pwd"
      path = path.trim()
      dockerfile = path + "/Dockerfile"
      anchorefile = path + "/anchore_images"
    }

    stage('Build') {
      // Build the image and push it to a staging repository
      repotag = inputConfig['dockerRepository'] + ":${BUILD_NUMBER}"
      docker.withRegistry('', registryCredential) {
        app = docker.build(repotag)
        app.push()
      }
    }

    stage('Parallel') {
      parallel Test: {
        app.inside {
            sh 'echo "Dummy - tests passed"'
        }
      },
      Analyze: {
        writeFile file: anchorefile, text: inputConfig['dockerRegistryHostname'] + "/" + repotag + " " + dockerfile
        anchore name: anchorefile, annotations: [[key: 'added-by', value: 'jenkins']] , autoSubscribeTagUpdates: false, bailOnFail: false, engineRetries: '10000'
      }
    }

  } 

  finally {
    stage('Cleanup') {
      // Delete the docker image and clean up any allotted resources
      sh script: "docker rmi " + repotag
    }
  }
}
