pipeline {
  environment {
    image = "nswccd-cato-app"
    registry = "jshark2010/$image"
    registryCredential = 'dockerhub'
    dockerImage = ''
  }
  agent any
  // parameters {
  //   string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
  //
  //   text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
  //
  //   booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
  //
  //   choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
  //
  //   password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
  //   // python3 reporting/exportPoam.py -c https://twistlock-console.oceast.cloudmegalodon.us -u jonathan@clearshark.com -p clearshark123! -o All -id sha256:c87e9a853fe046f445a1250c62432127db8b8b79e24ce73d68f6e74f86f147ac -t images -m reporting/POAM_Export_Sample.xlsx -eu "Jonathan Spigler"'
  // }
  stages {
    // stage('Cloning Git') {
    //   steps {
    //     git 'git@github.com:jcspigler2010/c-ato.git'
    //   }
    // }
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build("$registry:$BUILD_NUMBER", "-f ./app/Dockerfile ./app")
          imageid = dockerImage.imageName()
        }
      }
    }
    stage('Scan image') {
      steps{
        prismaCloudScanImage ca: '',
        cert: '',
        dockerAddress: 'unix:///var/run/docker.sock',
        image: "$imageid",
        key: '',
        logLevel: 'info',
        podmanPath: '',
        project: '',
        resultsFile: "$image-$BUILD_NUMBER-prisma-cloud-scan-results.json",
        ignoreImageBuildTime:true
      }
    }
    stage('Publish results') {
      steps{
        prismaCloudPublish resultsFilePattern: "$image-$BUILD_NUMBER-prisma-cloud-scan-results.json"
      }
    }
    stage('Export POAM') {
      steps{
        sh 'python3 reporting/exportPoam-0.1.3.py -c https://twistlock-console.oceast.cloudmegalodon.us -u jonathan@clearshark.com -p clearshark123! -o "ATO-NSWCCD-CATO-APP" -id "" -t images -m reporting/POAM_Export_Sample.xlsx -eu "Jonathan Spigler" -a "jshark"'
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
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
  }
  post {
        always {
            archiveArtifacts artifacts: '*.xlsx', onlyIfSuccessful: true
        }
    }
}
