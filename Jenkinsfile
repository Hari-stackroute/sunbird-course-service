#!groovy

node('build-slave') {

   currentBuild.result = "SUCCESS"
   cleanWs()

   try {

      stage('Checkout') {
         checkout scm
         commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
         branch_name = sh(script: 'git name-rev --name-only HEAD | rev | cut -d "/" -f1| rev', returnStdout: true).trim()
         echo "Git Hash: "+commit_hash
         echo "branch_name: "+branch_name
      }

      stage('Build') {
        env.NODE_ENV = "build"
        print "Environment will be : ${env.NODE_ENV}"
        sh('git submodule update --init')
        sh('git submodule update --init --recursive --remote')
        sh 'git log -1'
        sh 'cat service/conf/routes | grep v2'
        sh 'mvn clean install -DskipTests=true '

      }

      stage('Unit Tests') {
        sh "mvn test '-Dtest=!%regex[io.opensaber.registry.client.*]' -DfailIfNoTests=false"
      }

      stage('Package') {
        dir ('service') {
          sh 'mvn play2:dist'
        }
        sh('chmod 777 ./build.sh')
        sh "./build.sh ${commit_hash} ${branch_name} ${env.NODE_NAME} ${hub_org}"
      }
      stage('ArchiveArtifacts'){
           archiveArtifacts "metadata.json"
        }
    }
    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }
}
